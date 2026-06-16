# Databricks Lineage Strategy — Tiered Authority Model

**Status:** Design — ready to implement  
**Stories:** S12a (column-level system tables), S12b (merge engine), S12c (tier-3 optimization)  
**Depends on:** S7 (table lineage resolver), S9c (query history miner), S10a (OpenLineage receiver) — all complete  
**Audience:** AI coding agent (Claude Code / Copilot with Claude Sonnet) + engineers

---

## 1. Problem Statement

Databricks does not expose a single authoritative lineage source. Lineage information is scattered across three sources, each capturing a different slice of reality with a different accuracy, coverage, and cost profile:

| Source | Captures | Accuracy | Column-level? | Cost |
|---|---|---|---|---|
| `system.access.table_lineage` | UC-governed table flows | Highest — Databricks computed it | ❌ table only | Low (table read) |
| `system.access.column_lineage` | UC-governed column flows | Highest | ✅ native | Low (table read) |
| OpenLineage Spark events | Spark job internals at runtime | High — runtime observed | ✅ with facets | Medium (per-job config) |
| `system.access.query_history` + SQL parse | SQL Editor / JDBC / ad-hoc | Medium — inferred from text | ⚠️ parseable | High (parse each query) |

The naive approach (used by Atlan and DataHub) treats these as priority fallbacks: "try source 1, fall back to source 2." This is wrong because the sources are authoritative for *different kinds* of lineage. A table-level edge from Tier 1 and a column-level edge from Tier 3 for the same flow are complementary, not competing.

This document specifies a **tiered authority model with confidence-weighted merge** that:
1. Leads with `system.access.column_lineage` — free, authoritative, column-level (under-used by competitors)
2. Merges edges from all sources by authority, not fallback
3. Scores every edge with a confidence value and tracks provenance
4. Skips expensive SQL parsing for queries Tier 1 already covers

---

## 2. The Tier Model

### 2.1 Tier Definitions

```
TIER 1 — Databricks-computed lineage (system.access.table_lineage + column_lineage)
    Authority:   ABSOLUTE (1.0) — Databricks observed the actual query execution
    Source:      system tables, read via SQL
    Granularity: table_lineage → table edges; column_lineage → column edges
    Coverage:    All UC-governed query paths (notebooks, jobs, DLT, SQL warehouse)
    Parsing:     NONE — pre-computed by Databricks
    Cost:        LOW — two table reads per crawl window

TIER 2 — Runtime-observed lineage (OpenLineage Spark events)
    Authority:   HIGH (0.9) — observed at execution time by the OpenLineage integration
    Source:      push events to the S10a receiver
    Granularity: table + column (when columnLineage facet present)
    Coverage:    Spark jobs configured with OpenLineage, including non-UC paths
    Parsing:     NONE — structured event payload
    Cost:        MEDIUM — requires per-job OpenLineage configuration

TIER 3 — Parser-derived lineage (query_history + sqlglot)
    Authority:   MEDIUM (0.5–0.8) — inferred from SQL text, not execution
    Source:      system.access.query_history, parsed with sqlglot
    Granularity: table + column (when sqlglot resolves it)
    Coverage:    SQL Editor, JDBC/ODBC, ad-hoc queries NOT in Tier 1
    Parsing:     sqlglot — confidence varies by SQL complexity
    Cost:        HIGH — parse every uncovered query
```

### 2.2 Why Authority-Weighted, Not Fallback

A fallback model: "if Tier 1 has no edge for table X, try Tier 2, then Tier 3."

This loses information. Consider:
- Tier 1 `table_lineage` says `trades → positions` (table-level, authority 1.0)
- Tier 3 sqlglot says `trades.qty → positions.market_value` (column-level, authority 0.6)

A fallback model would pick only one. The correct model keeps both — the table edge is ground truth, the column edge enriches it. They merge into a complete picture: an authoritative table flow with inferred column detail.

The model is **merge-with-authority-resolution**: every edge from every tier is stored, keyed, and merged. When two edges have the same key, the higher-authority tier wins the core attributes, but provenance from all contributing tiers is preserved.

---

## 3. The Edge Data Model

### 3.1 LineageEdge — Internal Representation

Every tier produces edges in this common shape (extend the existing `LineageEdge` in `lineage/sql_parser.py`):

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import IntEnum


class LineageTier(IntEnum):
    """Lower value = higher authority. Used for merge resolution."""
    SYSTEM_TABLE = 1      # system.access.table_lineage / column_lineage
    OPENLINEAGE = 2       # OpenLineage Spark events
    QUERY_PARSE = 3       # query_history + sqlglot


@dataclass(frozen=True)
class LineageEdge:
    """A single lineage edge from any tier.

    Table-level edge: source_column and target_column are both None.
    Column-level edge: both are populated.
    """
    source_catalog: str
    source_schema: str
    source_table: str
    target_catalog: str
    target_schema: str
    target_table: str
    source_column: str | None = None      # None = table-level edge
    target_column: str | None = None      # None = table-level edge
    tier: LineageTier = LineageTier.QUERY_PARSE
    confidence: float = 0.5
    query_type: str = ""                  # INSERT, MERGE, CTAS, etc.
    statement_id: str = ""                # Databricks statement_id when available
    event_time: datetime | None = None

    @property
    def is_column_level(self) -> bool:
        return self.source_column is not None and self.target_column is not None

    @property
    def edge_key(self) -> tuple[str, ...]:
        """Deduplication key. Column edges and table edges have different keys."""
        return (
            f"{self.source_catalog}.{self.source_schema}.{self.source_table}".lower(),
            f"{self.target_catalog}.{self.target_schema}.{self.target_table}".lower(),
            (self.source_column or "").lower(),
            (self.target_column or "").lower(),
        )
```

### 3.2 CatalogLineageEvent — Output Document (Extended)

Extend the existing `CatalogLineageEvent` model with merge metadata:

```python
@dataclass
class CatalogLineageEvent:
    # ... existing fields ...
    event_id: str
    source_jrn: str
    target_jrn: str
    source_column: str | None = None       # NEW — for column-level
    target_column: str | None = None       # NEW — for column-level
    is_column_level: bool = False
    query_type: str = ""
    event_timestamp: datetime = ...

    # NEW — merge metadata
    tier: int = 3                          # winning tier (1, 2, or 3)
    confidence: float = 0.5                # winning edge confidence
    provenance: list[str] = field(default_factory=list)
        # e.g. ["system.access.column_lineage"]
    corroborated_by: list[str] = field(default_factory=list)
        # other tiers that independently found this edge
    first_seen: datetime = ...
    last_confirmed: datetime = ...
```

---

## 4. Confidence Scoring

### 4.1 Tier 1 — Always 1.0

System table lineage is computed by Databricks from actual execution. No uncertainty.

```python
TIER1_CONFIDENCE = 1.0
```

### 4.2 Tier 2 — Always 0.9

OpenLineage events are runtime-observed but depend on correct per-job configuration and the OpenLineage Spark integration version. Slightly below Tier 1 because the integration can miss edges in complex Spark plans.

```python
TIER2_CONFIDENCE = 0.9
```

### 4.3 Tier 3 — Parse-Complexity-Aware (0.2–0.8)

Confidence depends on how cleanly sqlglot could resolve the SQL. The more dynamic or opaque the SQL, the lower the confidence.

```python
import sqlglot
from sqlglot import exp


def score_parse_confidence(parsed: exp.Expression) -> float:
    """Assign a confidence score to a parsed SQL statement.

    Returns 0.2 (opaque) to 0.8 (clean projection).
    """
    # Dynamic SQL — cannot statically resolve
    if _has_dynamic_sql(parsed):
        return 0.2

    # SELECT * — column lineage cannot be resolved, table lineage is fine
    if _has_star_projection(parsed):
        return 0.3

    # Complex derived expressions (functions, case, window)
    if _has_complex_expressions(parsed):
        return 0.6

    # Simple column projection — SELECT a, b, c FROM t
    if _has_simple_projection(parsed):
        return 0.8

    # Default — joins, subqueries, resolvable but non-trivial
    return 0.5


def _has_dynamic_sql(parsed: exp.Expression) -> bool:
    """EXECUTE IMMEDIATE, dynamic table names, etc."""
    # sqlglot represents these as Command or unparseable nodes
    return any(isinstance(n, exp.Command) for n in parsed.walk())


def _has_star_projection(parsed: exp.Expression) -> bool:
    """SELECT * — no resolvable column lineage."""
    return any(isinstance(n, exp.Star) for n in parsed.walk())


def _has_complex_expressions(parsed: exp.Expression) -> bool:
    """Functions, CASE, window functions in projection."""
    select = parsed.find(exp.Select)
    if not select:
        return False
    for projection in select.expressions:
        if projection.find(exp.Func, exp.Case, exp.Window):
            return True
    return False


def _has_simple_projection(parsed: exp.Expression) -> bool:
    """Only bare column references in SELECT."""
    select = parsed.find(exp.Select)
    if not select:
        return False
    return all(
        isinstance(p, (exp.Column, exp.Alias)) and p.find(exp.Func) is None
        for p in select.expressions
    )
```

### 4.4 Corroboration Boost

When the same edge is found independently by multiple tiers, confidence increases. An edge confirmed by both Tier 1 and Tier 3 is more trustworthy than either alone.

```python
def apply_corroboration(base_confidence: float, corroborating_tiers: int) -> float:
    """Boost confidence when multiple tiers independently agree.

    Each additional corroborating tier adds a diminishing boost,
    capped at 1.0.
    """
    boost = 0.05 * corroborating_tiers
    return min(1.0, base_confidence + boost)
```

---

## 5. The Merge Engine

### 5.1 Algorithm

```
INPUT:  edges from Tier 1, Tier 2, Tier 3 (each a list of LineageEdge)
OUTPUT: merged, deduplicated, confidence-scored CatalogLineageEvent list

ALGORITHM:

  merged = {}   # edge_key → MergedEdge

  for tier in [TIER_1, TIER_2, TIER_3]:        # process in authority order
      for edge in tier.edges:
          key = edge.edge_key

          if key not in merged:
              # First time seeing this edge
              merged[key] = MergedEdge(
                  edge=edge,
                  winning_tier=edge.tier,
                  confidence=edge.confidence,
                  provenance=[edge.source_name],
                  corroborated_by=[],
                  first_seen=edge.event_time,
                  last_confirmed=edge.event_time,
              )
          else:
              existing = merged[key]

              if edge.tier < existing.winning_tier:
                  # Higher authority — this edge wins the core attributes
                  existing.corroborated_by.append(existing.winning_source)
                  existing.edge = edge
                  existing.winning_tier = edge.tier
                  existing.confidence = edge.confidence
                  existing.provenance.insert(0, edge.source_name)
              elif edge.tier == existing.winning_tier:
                  # Same tier — keep most recent, note corroboration
                  if edge.event_time > existing.last_confirmed:
                      existing.last_confirmed = edge.event_time
                  if edge.source_name not in existing.provenance:
                      existing.provenance.append(edge.source_name)
              else:
                  # Lower authority — existing wins, but record corroboration
                  existing.corroborated_by.append(edge.source_name)

              # Update timestamps
              if edge.event_time and (not existing.first_seen or edge.event_time < existing.first_seen):
                  existing.first_seen = edge.event_time
              if edge.event_time and edge.event_time > existing.last_confirmed:
                  existing.last_confirmed = edge.event_time

  # Apply corroboration boost
  for merged_edge in merged.values():
      n_corroborating = len(set(merged_edge.corroborated_by))
      merged_edge.confidence = apply_corroboration(
          merged_edge.confidence, n_corroborating
      )

  # Convert to CatalogLineageEvent
  return [to_catalog_lineage_event(m) for m in merged.values()]
```

### 5.2 The Critical Tier-3 Skip Optimization

Before Tier 3 parses any query, it checks whether Tier 1 already produced lineage for that `statement_id`. If so, parsing is skipped — Tier 1's authoritative lineage already covers it.

```python
def build_covered_statement_ids(tier1_edges: list[LineageEdge]) -> set[str]:
    """Collect statement_ids that Tier 1 already produced lineage for."""
    return {
        edge.statement_id
        for edge in tier1_edges
        if edge.statement_id
    }


def tier3_mine_with_skip(
    query_history_rows: Iterable[dict],
    covered_statement_ids: set[str],
) -> Iterator[LineageEdge]:
    """Parse only queries that Tier 1 did NOT already cover."""
    for row in query_history_rows:
        statement_id = row.get("statement_id", "")

        # SKIP — Tier 1 already has authoritative lineage for this query
        if statement_id in covered_statement_ids:
            continue

        # Only parse the uncovered queries (SQL Editor, JDBC, ad-hoc)
        sql_text = str(row.get("statement_text") or "")
        parsed = _safe_parse(sql_text)        # sqlglot
        del sql_text                          # REDACTION — discard immediately

        if parsed is None:
            continue

        confidence = score_parse_confidence(parsed)
        yield from _extract_edges(parsed, confidence, statement_id, row)
```

On a typical workspace, Tier 1 covers the large majority of queries (anything UC-governed). Tier 3 only parses the small fraction that bypassed UC tracking — slashing parse cost dramatically versus parsing every query.

---

## 6. Component Architecture

### 6.1 New and Modified Files

```
src/datacompass_metadata_harvester/
    lineage/
        resolver.py                    ← MODIFY (S12a) — add column_lineage reading
        column_resolver.py             ← NEW (S12a) — system.access.column_lineage
        databricks_miner.py            ← MODIFY (S12c) — skip optimization + column parse
        sql_parser.py                  ← MODIFY (S12c) — add column-level extraction
        merge_engine.py                ← NEW (S12b) — the merge algorithm
        confidence.py                  ← NEW (S12b) — scoring functions
        openlineage_normalizer.py      ← MODIFY (S12b) — extract column facets
    models/
        shared.py                      ← MODIFY — LineageTier enum, LineageEdge extension
        catalog.py                     ← MODIFY — CatalogLineageEvent merge fields
    pipeline/
        lineage_pipeline.py            ← MODIFY — wire merge engine into Databricks lineage

tests/
    lineage/
        test_column_resolver.py        ← NEW
        test_merge_engine.py           ← NEW
        test_confidence.py             ← NEW
        test_tier3_skip.py             ← NEW
        test_column_sql_parser.py      ← NEW
```

### 6.2 Data Flow

```
┌────────────────────────────────────────────────────────────┐
│ DatabricksLineagePipeline.run()                            │
│                                                            │
│  1. TIER 1 — table_lineage                                 │
│     resolver.get_table_lineage(since) → list[LineageEdge]  │
│                                                            │
│  2. TIER 1 — column_lineage                                │
│     column_resolver.get_column_lineage(since) → edges      │
│                                                            │
│  3. Build covered_statement_ids from Tier 1 edges          │
│                                                            │
│  4. TIER 3 — query_history (with skip)                     │
│     miner.get_lineage_events(since, covered_ids) → edges   │
│                                                            │
│  5. TIER 2 — already in MongoDB from OpenLineage receiver  │
│     merge_engine reads recent Tier-2 edges from MongoDB    │
│                                                            │
│  6. MERGE                                                  │
│     merge_engine.merge([tier1, tier2, tier3]) → events     │
│                                                            │
│  7. LOAD                                                   │
│     loader.upsert_lineage_event(event) for each            │
│                                                            │
│  8. WATERMARK                                              │
│     watermark_store.set(now)  ← lineage_watermarks         │
└────────────────────────────────────────────────────────────┘
```

Note: Tier 2 (OpenLineage) events arrive asynchronously via the S10a receiver and are already in MongoDB. The merge engine reads recent Tier-2 edges from the `lineage_events` collection (filtered by `tier == 2` and recent timestamp) and includes them in the merge. This means the Databricks lineage pipeline reconciles push-based Tier-2 edges with pull-based Tier-1/Tier-3 edges on each run.

---

## 7. Implementation Detail Per Component

### 7.1 S12a — Column-Level System Table Resolver

**File:** `lineage/column_resolver.py`

```python
"""Read column-level lineage from system.access.column_lineage.

This is Tier 1 — the highest authority, pre-computed by Databricks,
at column granularity, with NO SQL parsing required.

This is the single highest-value, lowest-effort lineage source and
the foundation of the tiered model.
"""
from __future__ import annotations

from collections.abc import Iterator
from datetime import datetime
from typing import Any

from datacompass_metadata_harvester.models.shared import (
    LineageEdge,
    LineageTier,
)
from datacompass_metadata_harvester.observability.logging import get_logger

_log = get_logger(__name__)

_COLUMN_LINEAGE_QUERY = """
SELECT
    source_table_catalog,
    source_table_schema,
    source_table_name,
    source_column_name,
    target_table_catalog,
    target_table_schema,
    target_table_name,
    target_column_name,
    entity_type,
    entity_run_id,
    event_time
FROM system.access.column_lineage
WHERE event_time > ?
  AND source_table_name IS NOT NULL
  AND target_table_name IS NOT NULL
ORDER BY event_time ASC
LIMIT ?
"""


class ColumnLineageResolver:
    """Reads system.access.column_lineage and produces Tier-1 column edges.

    Uses the existing Databricks connection infrastructure (same auth,
    same SQL warehouse) as the S7 table lineage resolver.
    """

    def __init__(self, query_client: Any) -> None:
        # query_client executes SQL and returns rows as dicts.
        # Same injectable pattern as S9c _MinerQueryClient.
        self._client = query_client

    def get_column_lineage(
        self,
        since: datetime,
        batch_size: int = 1000,
    ) -> Iterator[LineageEdge]:
        """Yield Tier-1 column-level lineage edges since the watermark.

        Each row in column_lineage is already a column→column edge.
        No parsing. Confidence is always 1.0.
        entity_run_id maps to statement_id for the Tier-3 skip optimization.
        """
        rows = self._client.query(
            _COLUMN_LINEAGE_QUERY,
            (since, batch_size),
        )
        for row in rows:
            yield LineageEdge(
                source_catalog=row["source_table_catalog"],
                source_schema=row["source_table_schema"],
                source_table=row["source_table_name"],
                source_column=row["source_column_name"],
                target_catalog=row["target_table_catalog"],
                target_schema=row["target_table_schema"],
                target_table=row["target_table_name"],
                target_column=row["target_column_name"],
                tier=LineageTier.SYSTEM_TABLE,
                confidence=1.0,
                statement_id=row.get("entity_run_id", ""),
                event_time=row.get("event_time"),
            )
```

**Effort: S (2–3 days)** — mirrors the existing S7 table lineage resolver pattern exactly.

---

### 7.2 S12c — Column-Level SQL Parsing

**File:** `lineage/sql_parser.py` (extend)

sqlglot has a dedicated column-lineage function. Extend `parse_lineage` to optionally return column edges:

```python
from sqlglot.lineage import lineage as sqlglot_column_lineage
from sqlglot import exp


def parse_column_lineage(
    sql: str,
    dialect: str,
    target_columns: list[str],
    default_catalog: str = "",
    default_schema: str = "",
) -> list[LineageEdge]:
    """Extract column-level lineage for the given target columns.

    For each target column, sqlglot traces it back through the query
    to its source column(s). Handles joins, CTEs, subqueries.

    Returns a list of column-level LineageEdge objects.
    Confidence is assigned by the caller based on parse complexity.

    The raw `sql` string must never be logged or stored — only
    redact_sql() output on the failure path.
    """
    edges: list[LineageEdge] = []
    try:
        for target_col in target_columns:
            node = sqlglot_column_lineage(
                target_col,
                sql,
                dialect=dialect,
            )
            # Walk the lineage tree to find leaf source columns
            for leaf in _walk_to_source_columns(node):
                edges.append(_build_column_edge(
                    leaf, target_col, default_catalog, default_schema,
                ))
    except Exception as exc:
        _log.warning(
            "column_lineage_parse_failed",
            redacted_sql=redact_sql(sql),
            error=str(exc),
        )
        return []
    return edges


def extract_target_columns(parsed: exp.Expression) -> list[str]:
    """Find the column names being written by an INSERT/MERGE/CTAS."""
    # INSERT INTO t (col1, col2) → ["col1", "col2"]
    # CREATE TABLE t AS SELECT a, b → ["a", "b"]
    ...
```

**Effort: M (1 sprint)** — sqlglot does the heavy lifting; the work is wiring and confidence scoring.

---

### 7.3 S12c — Tier-3 Miner with Skip Optimization

**File:** `lineage/databricks_miner.py` (modify)

Add the `covered_statement_ids` parameter to skip Tier-1-covered queries:

```python
def get_lineage_events(
    self,
    since: datetime,
    batch_size: int = 500,
    covered_statement_ids: set[str] | None = None,  # NEW
) -> Iterator[ExtractionResult[RawLineageEvent]]:
    """Mine query history, skipping queries Tier 1 already covered.

    covered_statement_ids: statement_ids that table_lineage/column_lineage
    already produced authoritative lineage for. These are skipped entirely —
    no parsing, no compute.
    """
    if not self._lineage_scope.enabled:
        return

    covered = covered_statement_ids or set()
    rows = self._client.query(_QUERY_HISTORY_SQL, (since, batch_size))

    for row in rows:
        statement_id = str(row.get("statement_id", ""))

        # SKIP — Tier 1 already has authoritative lineage
        if statement_id in covered:
            continue

        # Parse only uncovered queries (SQL Editor, JDBC, ad-hoc)
        statement_text = str(row.get("statement_text") or "")
        parsed = _safe_parse(statement_text, dialect="databricks")

        if parsed is not None:
            confidence = score_parse_confidence(parsed)
            # table-level edges
            table_edges = _extract_table_edges(parsed, confidence, statement_id, row)
            # column-level edges
            target_cols = extract_target_columns(parsed)
            col_edges = parse_column_lineage(
                statement_text, "databricks", target_cols,
            )
            for edge in [*table_edges, *col_edges]:
                yield ExtractionResult(record=_to_raw_event(edge))

        del statement_text  # REDACTION — discard immediately
```

**Effort: M (1 sprint)** — skip logic is simple; column parsing integration is the bulk.

---

### 7.4 S12b — The Merge Engine

**File:** `lineage/merge_engine.py` (new)

```python
"""Lineage merge engine — combines edges from all three tiers.

Authority-weighted merge: lower tier number = higher authority.
Same-key edges from multiple tiers are merged, not replaced —
the highest-authority tier wins core attributes, all tiers
contribute provenance, and corroboration boosts confidence.
"""
from __future__ import annotations

from dataclasses import dataclass, field
from datetime import datetime
from typing import Iterable

from datacompass_metadata_harvester.lineage.confidence import apply_corroboration
from datacompass_metadata_harvester.models.catalog import CatalogLineageEvent
from datacompass_metadata_harvester.models.shared import LineageEdge, LineageTier


_TIER_SOURCE_NAMES = {
    LineageTier.SYSTEM_TABLE: "system.access",
    LineageTier.OPENLINEAGE: "openlineage",
    LineageTier.QUERY_PARSE: "query_history",
}


@dataclass
class _MergedEdge:
    edge: LineageEdge
    winning_tier: LineageTier
    confidence: float
    provenance: list[str] = field(default_factory=list)
    corroborated_by: list[str] = field(default_factory=list)
    first_seen: datetime | None = None
    last_confirmed: datetime | None = None


class LineageMergeEngine:
    """Merges lineage edges from all tiers into deduplicated events."""

    def merge(
        self,
        tier_edges: Iterable[Iterable[LineageEdge]],
        jrn_resolver,  # resolves catalog.schema.table → JRN
    ) -> list[CatalogLineageEvent]:
        """Merge edges from multiple tiers.

        tier_edges: an iterable of edge collections, processed in the
        order given. Pass [tier1_edges, tier2_edges, tier3_edges] so
        higher-authority tiers are seen first.
        """
        merged: dict[tuple, _MergedEdge] = {}

        for edges in tier_edges:
            for edge in edges:
                self._absorb(merged, edge)

        # Apply corroboration boost
        for m in merged.values():
            n = len(set(m.corroborated_by))
            m.confidence = apply_corroboration(m.confidence, n)

        return [self._to_event(m, jrn_resolver) for m in merged.values()]

    def _absorb(self, merged: dict, edge: LineageEdge) -> None:
        key = edge.edge_key
        source_name = _TIER_SOURCE_NAMES[edge.tier]

        if key not in merged:
            merged[key] = _MergedEdge(
                edge=edge,
                winning_tier=edge.tier,
                confidence=edge.confidence,
                provenance=[source_name],
                first_seen=edge.event_time,
                last_confirmed=edge.event_time,
            )
            return

        existing = merged[key]

        if edge.tier < existing.winning_tier:
            # Higher authority wins core attributes
            existing.corroborated_by.append(
                _TIER_SOURCE_NAMES[existing.winning_tier]
            )
            existing.edge = edge
            existing.winning_tier = edge.tier
            existing.confidence = edge.confidence
            existing.provenance.insert(0, source_name)
        elif edge.tier == existing.winning_tier:
            if source_name not in existing.provenance:
                existing.provenance.append(source_name)
        else:
            # Lower authority corroborates
            existing.corroborated_by.append(source_name)

        # Timestamp reconciliation
        if edge.event_time:
            if existing.first_seen is None or edge.event_time < existing.first_seen:
                existing.first_seen = edge.event_time
            if existing.last_confirmed is None or edge.event_time > existing.last_confirmed:
                existing.last_confirmed = edge.event_time

    def _to_event(self, m: _MergedEdge, jrn_resolver) -> CatalogLineageEvent:
        e = m.edge
        source_jrn = jrn_resolver(e.source_catalog, e.source_schema, e.source_table)
        target_jrn = jrn_resolver(e.target_catalog, e.target_schema, e.target_table)

        # Deterministic event_id — column info included so column and
        # table edges for the same flow are distinct documents
        event_id = "|".join([
            source_jrn, target_jrn,
            e.source_column or "", e.target_column or "",
        ])

        return CatalogLineageEvent(
            event_id=event_id,
            source_jrn=source_jrn,
            target_jrn=target_jrn,
            source_column=e.source_column,
            target_column=e.target_column,
            is_column_level=e.is_column_level,
            query_type=e.query_type,
            event_timestamp=m.last_confirmed,
            tier=int(m.winning_tier),
            confidence=m.confidence,
            provenance=m.provenance,
            corroborated_by=list(set(m.corroborated_by)),
            first_seen=m.first_seen,
            last_confirmed=m.last_confirmed,
        )
```

**Effort: M (1 sprint)** — the algorithm is well-defined; the work is careful timestamp/provenance handling and tests.

---

### 7.5 S12b — OpenLineage Column Facet Extraction

**File:** `lineage/openlineage_normalizer.py` (modify)

OpenLineage events can carry a `columnLineage` facet. Extract it to produce Tier-2 column edges:

```python
def extract_column_edges(
    self,
    event: OpenLineageRunEvent,
) -> list[LineageEdge]:
    """Extract Tier-2 column edges from the OpenLineage columnLineage facet.

    The facet structure:
        outputs[].facets.columnLineage.fields[outputCol].inputFields[]
            = [{namespace, name (table), field (column)}]
    """
    edges = []
    for output in event.outputs:
        col_facet = output.facets.get("columnLineage", {})
        fields = col_facet.get("fields", {})
        for output_col, mapping in fields.items():
            for input_field in mapping.get("inputFields", []):
                edges.append(LineageEdge(
                    # resolve input_field namespace/name → catalog/schema/table
                    source_column=input_field["field"],
                    target_column=output_col,
                    tier=LineageTier.OPENLINEAGE,
                    confidence=0.9,
                    event_time=event.eventTime,
                    # ... resolve table parts from namespace
                ))
    return edges
```

**Effort: S (2–3 days)** — facet structure is well-documented in the OpenLineage spec.

---

## 8. MongoDB Schema

The `lineage_events` collection stores both table-level and column-level events, distinguished by `is_column_level`:

```json
// Table-level event (from Tier 1 table_lineage, corroborated by Tier 3)
{
  "event_id": "jrn:...trades|jrn:...positions||",
  "source_jrn": "jrn:jpm:ctcatalog::seal:34243::...:trading/trades",
  "target_jrn": "jrn:jpm:ctcatalog::seal:34243::...:risk/positions",
  "source_column": null,
  "target_column": null,
  "is_column_level": false,
  "query_type": "INSERT",
  "tier": 1,
  "confidence": 1.0,
  "provenance": ["system.access"],
  "corroborated_by": ["query_history"],
  "first_seen": "2025-05-01T06:00:00Z",
  "last_confirmed": "2025-06-01T06:00:00Z"
}

// Column-level event (from Tier 1 column_lineage)
{
  "event_id": "jrn:...trades|jrn:...positions|qty|market_value",
  "source_jrn": "jrn:jpm:ctcatalog::seal:34243::...:trading/trades",
  "target_jrn": "jrn:jpm:ctcatalog::seal:34243::...:risk/positions",
  "source_column": "qty",
  "target_column": "market_value",
  "is_column_level": true,
  "query_type": "INSERT",
  "tier": 1,
  "confidence": 1.0,
  "provenance": ["system.access.column_lineage"],
  "corroborated_by": [],
  "first_seen": "2025-05-01T06:00:00Z",
  "last_confirmed": "2025-06-01T06:00:00Z"
}
```

**Indexes:**
```python
db.lineage_events.create_index([("event_id", 1)], unique=True)
db.lineage_events.create_index([("source_jrn", 1)])
db.lineage_events.create_index([("target_jrn", 1)])
db.lineage_events.create_index([("is_column_level", 1)])
db.lineage_events.create_index([("tier", 1), ("confidence", -1)])
db.lineage_events.create_index([("last_confirmed", 1)])  # for Tier-2 recency reads
```

---

## 9. Consumer Query Patterns

The confidence/tier model lets consumers filter lineage by trust level:

```python
# High-confidence lineage only (Tier 1 + Tier 2)
db.lineage_events.find({"confidence": {"$gte": 0.9}})

# Column-level lineage for a specific table
db.lineage_events.find({
    "target_jrn": target,
    "is_column_level": True,
})

# Everything including inferred (all tiers)
db.lineage_events.find({"target_jrn": target})

# Lineage that needs review (low confidence, parser-derived only)
db.lineage_events.find({
    "tier": 3,
    "confidence": {"$lt": 0.5},
    "corroborated_by": {"$size": 0},
})
```

---

## 10. Story Breakdown

| Story | Title | Effort | Depends on |
|---|---|---|---|
| S12a | Column-level system table resolver | S (2–3 days) | S7 |
| S12b | Merge engine + confidence + OpenLineage column facets | M (1 sprint) | S12a, S10a |
| S12c | Tier-3 skip optimization + column SQL parsing | M (1 sprint) | S12a, S9c |

**Recommended order:** S12a → S12c → S12b.

S12a first because column_lineage is the highest-value, lowest-effort win and the foundation for everything. S12c next to add the skip optimization (immediate cost saving) and column parsing. S12b last to tie it all together with the merge engine — it needs the outputs of S12a and S12c to merge.

---

## 11. Validation — R6 Research Task

Before relying on Tier 1 column coverage, validate in the company environment:

**R6 — column_lineage coverage and lag:**

```sql
-- Run a known transformation
INSERT INTO test_schema.target_table (col_a, col_b)
SELECT s.col_x, s.col_y FROM test_schema.source_table s;

-- Wait, then check column_lineage
SELECT *
FROM system.access.column_lineage
WHERE target_table_name = 'target_table'
  AND event_time > current_timestamp() - INTERVAL 1 HOUR;
```

Measure:
1. **Does column_lineage populate at all?** (Some UC configurations don't enable it.)
2. **What's the lag?** (Time between query execution and lineage table population.)
3. **Coverage rate:** Run 10 different transformation patterns (joins, CTEs, aggregations) and count how many appear in column_lineage.

If coverage is high (>80%), Tier 1 carries column lineage and Tier 3 is a thin gap-filler. If coverage is low or lag is high, Tier 3 sqlglot column parsing becomes the primary column source and needs more investment.

---

## 12. Claude Code Prompts

### S12a Prompt

> Implement S12a — column-level lineage from `system.access.column_lineage`. Reference `docs/dbx_lineage_strategy.md` sections 3, 4.1, 7.1.
>
> Create `lineage/column_resolver.py` with `ColumnLineageResolver` following the exact pattern of the existing S7 table lineage resolver. Add the `LineageTier` IntEnum and extend `LineageEdge` (in `models/shared.py`) with `source_column`, `target_column`, `tier`, `confidence`, `statement_id`, `event_time`, the `is_column_level` property, and the `edge_key` property. Extend `CatalogLineageEvent` (in `models/catalog.py`) with the merge metadata fields from section 3.2.
>
> Query `system.access.column_lineage` using `?` placeholders (Databricks). Each row is a column→column edge with confidence 1.0, tier SYSTEM_TABLE. Map `entity_run_id` to `statement_id`.
>
> Add unit tests in `tests/lineage/test_column_resolver.py` — mock the query client, verify column edges are produced with tier 1, confidence 1.0, and correct edge_key. Run pytest and mypy green. No `--unsafe-fixes`.

### S12c Prompt

> Implement S12c — Tier-3 skip optimization and column-level SQL parsing. Reference `docs/dbx_lineage_strategy.md` sections 4.3, 5.2, 7.2, 7.3.
>
> Add `score_parse_confidence()` and the helper functions to `lineage/confidence.py`. Add `parse_column_lineage()` and `extract_target_columns()` to `lineage/sql_parser.py` using `sqlglot.lineage.lineage`. Modify `databricks_miner.py` `get_lineage_events()` to accept `covered_statement_ids: set[str]` and skip queries already in that set. After parsing, produce both table-level and column-level edges. Preserve the redaction invariant — `del statement_text` after parse, `redact_sql()` only on failure.
>
> Tests: `test_confidence.py` (scoring for star/dynamic/simple/complex SQL), `test_tier3_skip.py` (covered statement_ids are skipped), `test_column_sql_parser.py` (column edges through joins and CTEs). Run pytest and mypy green. No `--unsafe-fixes`.

### S12b Prompt

> Implement S12b — the lineage merge engine. Reference `docs/dbx_lineage_strategy.md` sections 5.1, 7.4, 7.5.
>
> Create `lineage/merge_engine.py` with `LineageMergeEngine.merge()` implementing the authority-weighted merge algorithm. Create `apply_corroboration()` in `lineage/confidence.py`. Modify `openlineage_normalizer.py` to extract Tier-2 column edges from the `columnLineage` facet. Modify `pipeline/lineage_pipeline.py` `DatabricksLineagePipeline` to wire the full flow from section 6.2: Tier 1 table + column, build covered_statement_ids, Tier 3 with skip, read recent Tier 2 from MongoDB, merge, load.
>
> Tests: `test_merge_engine.py` — higher tier wins core attributes, same-tier keeps recent, corroboration boosts confidence, table and column edges produce distinct event_ids. Run pytest and mypy green. No `--unsafe-fixes`.



# Workflow Count & Orchestration for Tiered Databricks Lineage

## The Short Answer
One scheduled workflow for the pull-based tiers (1 and 3 + merge), plus one always-on deployment for the push-based tier (2). The merge happens inside the scheduled workflow, not as a separate job.
1 CronJob   — databricks-lineage (Tier 1 + Tier 3 + Merge)
1 Deployment — openlineage-receiver (Tier 2, already running from S10a)

## Why Not Separate Workflows Per Tier
The instinct is to run Tier 1, Tier 2, and Tier 3 as three separate jobs. That's wrong for one critical reason: the skip optimization and the merge require the tiers to run together in sequence.
Tier 3 needs Tier 1's covered_statement_ids  → they must run in the same process
Merge needs all three tiers' edges            → merge must run after 1 and 3
If you split them into separate jobs, you'd need to pass covered_statement_ids between jobs (shared state in MongoDB), and the merge job would need to wait for both to finish (job dependencies). That's more orchestration complexity for no benefit — Tier 1 and Tier 3 both read from the same Databricks SQL Warehouse with the same connection, so running them in one process is cheaper and simpler.

## The Orchestration Model

Workflow 1 — databricks-lineage CronJob (Pull + Merge)
A single job that runs the full pull-based pipeline in sequence:
┌─────────────────────────────────────────────────────────┐
│  databricks-lineage CronJob (every hour)                │
│                                                         │
│  Step 1: TIER 1 — table_lineage                         │
│          resolver.get_table_lineage(since)              │
│          → tier1_table_edges                            │
│                                                         │
│  Step 2: TIER 1 — column_lineage                        │
│          column_resolver.get_column_lineage(since)      │
│          → tier1_column_edges                           │
│                                                         │
│  Step 3: Build covered_statement_ids from Tier 1        │
│                                                         │
│  Step 4: TIER 3 — query_history (with skip)             │
│          miner.get_lineage_events(since, covered_ids)   │
│          → tier3_edges (only uncovered queries parsed)  │
│                                                         │
│  Step 5: TIER 2 — read recent OpenLineage edges         │
│          merge_engine reads tier==2 events from MongoDB │
│          → tier2_edges                                  │
│                                                         │
│  Step 6: MERGE all three tiers                          │
│          merge_engine.merge([t1, t2, t3])               │
│                                                         │
│  Step 7: LOAD merged events to MongoDB                  │
│                                                         │
│  Step 8: Advance lineage_watermark                      │
└─────────────────────────────────────────────────────────┘
One process, one connection, one watermark. This is the DatabricksLineagePipeline.run() from section 6.2 of the design doc.
Workflow 2 — openlineage-receiver Deployment (Push)
Already running from S10a. It accepts OpenLineage Spark events continuously and writes Tier-2 edges to MongoDB as they arrive. The scheduled job reads these in Step 5.
┌─────────────────────────────────────────────────────────┐
│  openlineage-receiver Deployment (always on, 2 replicas)│
│                                                         │
│  Spark job emits event → POST /api/v1/lineage           │
│       → normalize → extract table + column edges        │
│       → write to MongoDB with tier=2, confidence=0.9    │
│                                                         │
│  Runs independently. No schedule. Event-driven.         │
└─────────────────────────────────────────────────────────┘

## Why Tier 2 Is Separate (and Must Be)
Tier 2 can't be folded into the scheduled job because OpenLineage is push-based — events arrive whenever Spark jobs run, not on a schedule. The receiver must be always-on to catch them. The scheduled job reconciles these already-stored Tier-2 edges with the freshly-pulled Tier-1/Tier-3 edges during merge.
This is the architectural split:
| Tier | Trigger | Workflow type | Why |
| Tier 1 | Schedule | CronJob | Pull from system tables on a cadence | 
| Tier 3 | Schedule | CronJob (same job) | Pull from query_history, needs Tier 1's skip set |
| Merge | Schedule | CronJob (same job)Needs Tier 1 + Tier 3 outputs |
| Tier 2 | Event | Deployment | Push-based, must be always-on |

## Schedule Design
```yaml
# k8s/cronjobs/databricks-lineage.yaml
schedule: "0 * * * *"          # hourly
concurrencyPolicy: Forbid       # never overlap runs
activeDeadlineSeconds: 2700     # 45 min ceiling
backoffLimit: 1
```

Why hourly: Tier 1 system tables have some population lag (validate exact lag with R6). Hourly catches lineage within an hour of query execution — fast enough for a catalog, not so frequent it wastes compute. The Tier-3 skip optimization keeps each run cheap because most queries are already covered by Tier 1.

Why Forbid concurrency: Two lineage runs overlapping would double-process the same watermark window and contend on the same MongoDB writes. The watermark model assumes serial runs.

## Comparison to the Crawler Workflows
Your existing setup already follows this pattern — lineage mirrors it:

| Workflow | Type | Schedule | What it does |
| --- | --- | --- | --- |
| databricks-crawler | CronJob | every 4h | Metadata inventory | 
| databricks-miner | CronJob | every 4h (offset :30) | Replaced by databricks-lineage |
| databricks-lineage | CronJob | hourly | NEW — Tier 1+3+merge |
| openlineage-receiver | Deployment | always-on | Tier 2 |

Important: databricks-lineage (the new tiered job) replaces the old databricks-miner (S9c standalone). The standalone miner becomes Step 4 inside the new pipeline. You don't run both — the tiered pipeline subsumes the miner.

## What Changes in Your K8s Manifests
```
k8s/cronjobs/
    databricks-crawler.yaml       ← unchanged
    databricks-miner.yaml         ← DELETE (subsumed into databricks-lineage)
    databricks-lineage.yaml       ← NEW (Tier 1 + 3 + merge)
    snowflake-crawler.yaml        ← unchanged
    snowflake-miner.yaml          ← unchanged (Snowflake stays single-tier for now)
    ...

k8s/deployments/
    openlineage-receiver.yaml     ← unchanged (already Tier 2)
```

Only Databricks gets the tiered treatment because only Databricks has all three sources (system.access.*, OpenLineage Spark, query_history). Snowflake and Oracle keep their single-tier miners until they have equivalent multi-source lineage.

## The Full Orchestration Picture

                     DATABRICKS LINEAGE ORCHESTRATION

  PUSH (always-on)                      PULL (scheduled hourly)
  ┌───────────────────┐                  ┌──────────────────────────┐
  │ Spark jobs        │                  │  databricks-lineage      │
  │   emit OL events  │                  │  CronJob (0 * * * *)     │
  └────────┬──────────┘                  │                          │
           │                             │  1. Tier 1 table_lineage │
           ▼                             │  2. Tier 1 column_lineage│
  ┌───────────────────┐                  │  3. build skip set       │
  │ openlineage-      │                  │  4. Tier 3 (skip-opt)    │
  │ receiver          │                  │  5. read Tier 2 ◄────────┼──┐
  │ Deployment        │                  │  6. MERGE                │  │
  │  tier=2, conf=0.9 │                  │  7. load merged events   │  │
  └────────┬──────────┘                  │  8. advance watermark    │  │
           │                             └──────────┬───────────────┘  │
           │                                        │                  │
           ▼                                        ▼                  │
  ┌─────────────────────────────────────────────────────────────────┐  │
  │                    MongoDB lineage_events                       │  │
  │   tier=2 edges (continuous) ────────────────────────────────────┼──┘
  │   tier=1,3 merged edges (hourly)                                │
  └─────────────────────────────────────────────────────────────────┘

The receiver writes Tier-2 edges continuously. The hourly job reads them back during merge and reconciles them with Tier-1/Tier-3 edges. One scheduled workflow, one always-on workflow, clean separation between push and pull.