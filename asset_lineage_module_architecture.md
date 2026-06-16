# Asset Details & Lineage Visualization — Architecture & Design

**Module:** Asset Explorer + Lineage Visualization  
**Integrates into:** Control Plane SPA (existing) + Workflow Management API (existing)  
**Data source:** MongoDB collections written by the Harvester  
**Status:** Design — ready to implement

---

## 1. System Context

### 1.1 Current Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  HARVESTER                                                      │
│  Crawlers + Miners → writes to MongoDB                          │
│  Collections: DataDistribution, DataAttribute, lineage_events,  │
│               crawler_watermarks, lineage_watermarks            │
└────────────────────────────────────┬────────────────────────────┘
                                     │ writes
                                     ▼
                              ┌──────────────┐
                              │   MongoDB     │
                              └──────┬───────┘
                                     │ reads
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│  WORKFLOW MANAGEMENT API (WMA)                                  │
│  FastAPI — workflow CRUD, run history, logs, config management  │
│  Reads: admin_workflows, admin_runs                             │
└────────────────────────────────────┬────────────────────────────┘
                                     │ REST API
                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│  CONTROL PLANE SPA                                              │
│  React — Dashboard, Workflow List, Run History, Config Editor   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Target Architecture (After This Work)

```
┌─────────────────────────────────────────────────────────────────┐
│  HARVESTER                                                      │
│  Crawlers + Miners → writes to MongoDB                          │
└────────────────────────────────────┬────────────────────────────┘
                                     │ writes
                                     ▼
                              ┌──────────────┐
                              │   MongoDB    │
                              └──┬────────┬──┘
                                 │        │
                     reads ┌─────┘        └──────┐ reads
                           ▼                     ▼
┌──────────────────────────────┐  ┌───────────────────────────────┐
│  WMA — Workflow Routers      │  │  WMA — Asset Routers (NEW)    │
│  /api/workflows              │  │  /api/assets                  │
│  /api/runs                   │  │  /api/assets/{jrn}            │
│  /api/config                 │  │  /api/assets/{jrn}/attributes │
│  /api/schedules              │  │  /api/lineage/graph           │
│  /api/dashboard/summary      │  │  /api/lineage/impact          │
└──────────────┬───────────────┘  └───────────────┬───────────────┘
               │                                  │
               └──────────┬───────────────────────┘
                          │ REST API
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│  CONTROL PLANE SPA                                               │
│                                                                  │
│  ┌───────────────────┐  ┌────────────────────────────────────┐   │
│  │ Workflow Module   │  │ Asset Explorer Module (NEW)        │   │
│  │ (existing)        │  │                                    │   │
│  │ Dashboard         │  │ Asset Catalog (search + browse)    │   │
│  │ Workflow List     │  │ Asset Detail (metadata + attrs)    │   │
│  │ Run History       │  │ Lineage Graph (React Flow)         │   │
│  │ Config Editor     │  │ Impact Analysis                    │   │
│  │ New Workflow      │  │ Column Lineage                     │   │
│  └───────────────────┘  └────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

The asset routers are added to the existing WMA service — not a new service. They read from the same MongoDB instance the Harvester writes to. No new infrastructure.

---

## 2. Design Decisions

### 2.1 Why Add to WMA, Not a Separate Service

| Option | Pros | Cons |
|---|---|---|
| New service | Clean separation | Extra deployment, extra infra, shared MongoDB anyway |
| Add to WMA | One deployment, shared DB client, shared auth | Slightly larger service |

The WMA already has a MongoDB connection, auth middleware, and health checks. Adding routers for asset/lineage reads is simpler than standing up a new service. If the WMA grows too large later, the routers can be extracted — they share no state with the workflow routers.

### 2.2 Read-Only from Harvester Collections

The asset/lineage routers **only read** from Harvester collections (`DataDistribution`, `DataAttribute`, `lineage_events`). They never write. This means:
- No schema coupling — if the Harvester changes a field, only the read-mapping in the router needs updating
- No write contention — the Harvester owns writes, the WMA owns reads
- No migration risk — adding these routers doesn't touch Harvester code

### 2.3 Graph Assembly at Query Time

Lineage is stored as edges. The graph is assembled on demand by the API via BFS traversal from a starting JRN. No pre-computed graph storage. Reasons:
- The graph is a view, not a source of truth
- Users query locally (N hops around one table), not globally
- Pre-computed graphs would need invalidation on every Harvester write

### 2.4 Layout Computed Client-Side

The API returns nodes with `position: {x: 0, y: 0}`. The SPA runs dagre for layout. This keeps the API stateless and lets the UI re-layout interactively on zoom/filter/expand.

---

## 3. API Design — Asset Routers

### 3.1 Asset Catalog — Search and Browse

```
GET /api/assets
    ?q=positions                    — full-text search on name
    &platform=DATABRICKS            — filter by platform
    &environment=PROD               — filter by environment
    &type=TABLE                     — filter by SourceTypeName
    &schema=risk                    — filter by schema/project
    &status=ACTIVE                  — filter by status
    &page=1&page_size=50            — pagination
    &sort=name                      — sort field
    &order=asc                      — sort order

Response:
{
    "total": 4518,
    "page": 1,
    "page_size": 50,
    "items": [
        {
            "jrn": "jrn:jpm:ctcatalog::seal:34243::...",
            "name": "positions",
            "platform": "DATABRICKS",
            "environment": "PROD",
            "type_name": "TABLE",
            "schema": "risk",
            "catalog": "34243_ctg_prod",
            "status": "ACTIVE",
            "owner": "risk-team@jpmc.com",
            "last_modified": "2025-06-01T06:00:00Z",
            "has_lineage": true,
            "lineage_edge_count": 12,
            "attribute_count": 42
        }
    ],
    "facets": {
        "platforms": [
            {"value": "DATABRICKS", "count": 3200},
            {"value": "SNOWFLAKE", "count": 850},
            {"value": "TABLEAU", "count": 120},
            {"value": "ORACLE", "count": 348}
        ],
        "environments": [
            {"value": "PROD", "count": 4000},
            {"value": "DEV", "count": 518}
        ],
        "types": [
            {"value": "TABLE", "count": 3800},
            {"value": "VIEW", "count": 600},
            {"value": "WORKFLOW", "count": 118}
        ]
    }
}
```

### 3.2 Asset Detail

```
GET /api/assets/{jrn}

Response:
{
    "jrn": "jrn:jpm:ctcatalog::seal:34243::...:risk/positions",
    "name": "positions",
    "display_name": "positions",
    "platform": "DATABRICKS",
    "environment": "PROD",
    "type_name": "TABLE",
    "status": "ACTIVE",
    "catalog": "34243_ctg_prod",
    "schema": "risk",
    "table": "positions",
    "qualified_name": "34243_ctg_prod.risk.positions",
    "owner": "risk-team@jpmc.com",
    "description": "Aggregated trading positions...",
    "last_modified": "2025-06-01T06:00:00Z",
    "last_crawled": "2025-06-01T06:12:34Z",
    "created_at": "2024-01-15T...",
    "seal_node_id": "34243",
    "properties": {},
    "source": { ... },
    "atlan": { ... },
    "lineage_summary": {
        "upstream_count": 5,
        "downstream_count": 3,
        "column_lineage_available": true
    }
}
```

### 3.3 Asset Attributes (Columns / Fields)

```
GET /api/assets/{jrn}/attributes
    ?page=1&page_size=100

Response:
{
    "total": 42,
    "items": [
        {
            "jrn": "jrn:...positions/trade_id",
            "name": "trade_id",
            "ordinal": 1,
            "data_type": "BIGINT",
            "nullable": false,
            "description": "Primary key — trade identifier",
            "has_column_lineage": true
        },
        {
            "jrn": "jrn:...positions/market_value",
            "name": "market_value",
            "ordinal": 2,
            "data_type": "DECIMAL(18,4)",
            "nullable": true,
            "description": null,
            "has_column_lineage": true
        }
    ]
}
```

### 3.4 Lineage Graph

```
GET /api/lineage/graph
    ?jrn={starting_jrn}
    &depth=2                        — hops in each direction
    &direction=both                 — upstream | downstream | both
    &min_confidence=0.0             — filter low-confidence edges
    &column_level=false             — table or column lineage
    &column={column_name}           — column lineage for specific column

Response:
{
    "root_jrn": "jrn:...risk/positions",
    "depth": 2,
    "direction": "both",
    "nodes": [
        {
            "id": "jrn:...trading/trades",
            "type": "tableNode",
            "position": {"x": 0, "y": 0},
            "data": {
                "label": "trades",
                "platform": "DATABRICKS",
                "schema": "34243_ctg_prod.trading",
                "typeName": "TABLE",
                "environment": "PROD",
                "status": "ACTIVE",
                "isRoot": false,
                "columns": [
                    {"name": "trade_id", "type": "BIGINT"},
                    {"name": "qty", "type": "INT"},
                    {"name": "cusip", "type": "VARCHAR(12)"}
                ]
            }
        },
        {
            "id": "jrn:...risk/positions",
            "type": "tableNode",
            "position": {"x": 0, "y": 0},
            "data": {
                "label": "positions",
                "platform": "DATABRICKS",
                "schema": "34243_ctg_prod.risk",
                "typeName": "TABLE",
                "environment": "PROD",
                "status": "ACTIVE",
                "isRoot": true,
                "columns": [
                    {"name": "position_id", "type": "BIGINT"},
                    {"name": "market_value", "type": "DECIMAL(18,4)"}
                ]
            }
        }
    ],
    "edges": [
        {
            "id": "jrn:...trades__jrn:...positions",
            "source": "jrn:...trading/trades",
            "target": "jrn:...risk/positions",
            "animated": false,
            "data": {
                "tier": 1,
                "confidence": 1.0,
                "queryType": "INSERT",
                "provenance": ["system.access"],
                "corroboratedBy": ["query_history"]
            },
            "style": {
                "stroke": "#16a34a",
                "strokeWidth": 2
            }
        }
    ],
    "column_edges": [
        {
            "id": "jrn:...trades:qty__jrn:...positions:market_value",
            "source": "jrn:...trading/trades",
            "sourceHandle": "qty",
            "target": "jrn:...risk/positions",
            "targetHandle": "market_value",
            "data": {
                "tier": 1,
                "confidence": 1.0
            }
        }
    ],
    "stats": {
        "total_nodes": 8,
        "total_edges": 12,
        "total_column_edges": 34,
        "tiers": {"1": 8, "2": 2, "3": 2}
    }
}
```

### 3.5 Impact Analysis

```
GET /api/lineage/impact
    ?jrn={starting_jrn}
    &depth=5                        — how many hops downstream
    &min_confidence=0.5

Response:
{
    "root_jrn": "jrn:...trading/trades",
    "impacted_assets": [
        {
            "jrn": "jrn:...risk/positions",
            "name": "positions",
            "platform": "DATABRICKS",
            "distance": 1,
            "path": ["jrn:...trades", "jrn:...positions"],
            "min_confidence": 1.0,
            "impact_type": "direct"
        },
        {
            "jrn": "jrn:...reports/risk_dashboard",
            "name": "risk_dashboard",
            "platform": "TABLEAU",
            "distance": 2,
            "path": ["jrn:...trades", "jrn:...positions", "jrn:...risk_dashboard"],
            "min_confidence": 0.9,
            "impact_type": "indirect"
        }
    ],
    "total_impacted": 7,
    "max_distance": 3
}
```

---

## 4. SPA Module Design

### 4.1 Navigation Structure

```
Control Plane SPA
├── Workflows (existing)
│   ├── Dashboard
│   ├── Workflow List
│   ├── Run History
│   └── Config Editor
│
└── Assets (NEW)
    ├── Asset Catalog          /assets
    ├── Asset Detail           /assets/:jrn
    │   ├── Overview tab
    │   ├── Attributes tab
    │   └── Lineage tab
    └── Impact Analysis        /assets/:jrn/impact
```

### 4.2 Page Specifications

#### Asset Catalog Page (`/assets`)

```
┌──────────────────────────────────────────────────────────────────┐
│  Assets                                                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ 🔍 Search assets...                                        │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Filters                                                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │Platform ▼│ │Type    ▼ │ │Status  ▼ │ │Env     ▼ │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                  │
│  4,518 assets                                       Page 1 of 91│
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ ⬡ positions              TABLE    DATABRICKS   PROD        │  │
│  │   34243_ctg_prod.risk    12 lineage edges   42 columns     │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │ ⬡ trades                 TABLE    DATABRICKS   PROD        │  │
│  │   34243_ctg_prod.trading  8 lineage edges   28 columns     │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │ ⬡ risk_dashboard         DATASOURCE TABLEAU    PROD        │  │
│  │   kfnawaz-site/Risk       3 lineage edges   15 fields     │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ◄ 1 2 3 4 5 ... 91 ►                                           │
└──────────────────────────────────────────────────────────────────┘
```

#### Asset Detail Page (`/assets/:jrn`)

Three tabs — Overview, Attributes, Lineage:

```
┌──────────────────────────────────────────────────────────────────┐
│  ← Assets / positions                                            │
│  34243_ctg_prod.risk.positions                                   │
│  TABLE • DATABRICKS • PROD • ACTIVE                              │
├──────────────────────────────────────────────────────────────────┤
│  [Overview]  [Attributes (42)]  [Lineage]                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  OVERVIEW TAB                                                    │
│                                                                  │
│  Metadata                                                        │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ Owner:          risk-team@jpmc.com                          │  │
│  │ Catalog:        34243_ctg_prod                              │  │
│  │ Schema:         risk                                        │  │
│  │ SEAL Node:      34243                                       │  │
│  │ Last Modified:  Jun 1, 2025 06:00 UTC                       │  │
│  │ Last Crawled:   Jun 1, 2025 06:12 UTC                       │  │
│  │ Created:        Jan 15, 2024                                │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Lineage Summary                                                 │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ 5 upstream tables → [positions] → 3 downstream consumers   │  │
│  │ Column lineage available ✓                                  │  │
│  │ [View Full Lineage]  [Impact Analysis]                      │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

#### Attributes Tab

```
┌──────────────────────────────────────────────────────────────────┐
│  [Overview]  [Attributes (42)]  [Lineage]                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ #   Name             Type           Nullable  Lineage      │  │
│  │ ─── ──────────────── ────────────── ───────── ───────      │  │
│  │ 1   trade_id         BIGINT         No        ⇄ 3 edges   │  │
│  │ 2   market_value     DECIMAL(18,4)  Yes       ⇄ 2 edges   │  │
│  │ 3   cusip            VARCHAR(12)    No        ⇄ 1 edge    │  │
│  │ 4   position_date    DATE           No        —            │  │
│  │ 5   book_id          INT            Yes       ⇄ 1 edge    │  │
│  │ ...                                                         │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Click column name to view column lineage                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

#### Lineage Tab

```
┌──────────────────────────────────────────────────────────────────┐
│  [Overview]  [Attributes (42)]  [Lineage]                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Controls                                                        │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ Direction: [Both ▼]  Depth: [2 ▼]  Min Confidence: [0 ▼]  │  │
│  │ View: [○ Table  ● Column]  [🔍 Fit] [↻ Reset]             │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │   ┌───────────┐    ┌───────────────┐    ┌────────────┐    │  │
│  │   │ trades    │    │  positions ★  │    │ risk_dash  │    │  │
│  │   │ DATABRICKS│───▶│  DATABRICKS   │───▶│ TABLEAU    │    │  │
│  │   └───────────┘    └───────────────┘    └────────────┘    │  │
│  │        ▲                                                   │  │
│  │   ┌───────────┐                                            │  │
│  │   │ prices    │                                            │  │
│  │   │ SNOWFLAKE │                                            │  │
│  │   └───────────┘                                            │  │
│  │                                                            │  │
│  │  Legend: ━━ Tier 1 (conf 1.0)  ── Tier 2 (0.9)            │  │
│  │          ╌╌ Tier 3 (< 0.5)    ★ = root asset              │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Edge Detail (on click)                                          │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ trades → positions                                          │  │
│  │ Tier: 1 (system.access)  Confidence: 1.0                   │  │
│  │ Query type: INSERT                                          │  │
│  │ Corroborated by: query_history                              │  │
│  │ First seen: May 1, 2025  Last confirmed: Jun 1, 2025       │  │
│  │ [View Column Lineage]                                       │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. Graph Assembly — Server-Side Implementation

### 5.1 BFS Traversal Algorithm

```python
"""Graph assembly service — builds React Flow graph from lineage_events.

BFS traversal from a starting JRN, N hops out.
Collects edges, deduplicates, enriches nodes from DataDistribution.
Returns React Flow-shaped JSON.
"""
from collections import deque
from datetime import datetime, UTC
from typing import Any

from pymongo.database import Database


class LineageGraphService:
    """Assembles lineage graphs from MongoDB lineage_events."""

    def __init__(self, db: Database) -> None:
        self._db = db

    async def build_graph(
        self,
        start_jrn: str,
        depth: int = 2,
        direction: str = "both",
        min_confidence: float = 0.0,
        column_level: bool = False,
    ) -> dict[str, Any]:
        visited: set[str] = set()
        edge_list: list[dict] = []
        queue: deque[tuple[str, int]] = deque([(start_jrn, 0)])

        while queue:
            current, hop = queue.popleft()
            if current in visited:
                continue
            visited.add(current)
            if hop >= depth:
                continue

            base_filter: dict = {
                "is_column_level": column_level,
                "confidence": {"$gte": min_confidence},
            }

            if direction in ("downstream", "both"):
                cursor = self._db.lineage_events.find(
                    {**base_filter, "source_jrn": current}
                )
                for edge in cursor:
                    edge_list.append(edge)
                    if edge["target_jrn"] not in visited:
                        queue.append((edge["target_jrn"], hop + 1))

            if direction in ("upstream", "both"):
                cursor = self._db.lineage_events.find(
                    {**base_filter, "target_jrn": current}
                )
                for edge in cursor:
                    edge_list.append(edge)
                    if edge["source_jrn"] not in visited:
                        queue.append((edge["source_jrn"], hop + 1))

        nodes = await self._enrich_nodes(visited, start_jrn, column_level)
        edges = self._format_edges(edge_list, column_level)
        column_edges = self._format_column_edges(edge_list) if column_level else []

        return {
            "root_jrn": start_jrn,
            "depth": depth,
            "direction": direction,
            "nodes": nodes,
            "edges": edges,
            "column_edges": column_edges,
            "stats": {
                "total_nodes": len(nodes),
                "total_edges": len(edges),
                "total_column_edges": len(column_edges),
                "tiers": self._count_by_tier(edge_list),
            },
        }

    async def _enrich_nodes(
        self,
        jrns: set[str],
        root_jrn: str,
        include_columns: bool,
    ) -> list[dict]:
        nodes = []
        for dist in self._db.data_distributions.find(
            {"jrn": {"$in": list(jrns)}}
        ):
            node = {
                "id": dist["jrn"],
                "type": "columnTableNode" if include_columns else "tableNode",
                "position": {"x": 0, "y": 0},
                "data": {
                    "label": dist["name"],
                    "platform": dist["platform"],
                    "schema": dist["source"]["sourceQualifiedName"],
                    "typeName": dist["source"]["sourceTypeName"],
                    "environment": dist.get("environment", ""),
                    "status": dist["source"]["sourceStatus"],
                    "isRoot": dist["jrn"] == root_jrn,
                },
            }
            if include_columns:
                attrs = list(self._db.data_attributes.find(
                    {"parent_jrn": dist["jrn"]},
                ).sort("ordinal", 1))
                node["data"]["columns"] = [
                    {"name": a["name"], "type": a.get("raw_data_type", "")}
                    for a in attrs
                ]
            nodes.append(node)
        return nodes

    def _format_edges(self, raw: list[dict], column_level: bool) -> list[dict]:
        seen: set[str] = set()
        edges = []
        for e in raw:
            eid = f"{e['source_jrn']}__{e['target_jrn']}"
            if column_level:
                eid += f"__{e.get('source_column', '')}__{e.get('target_column', '')}"
            if eid in seen:
                continue
            seen.add(eid)
            conf = e.get("confidence", 0.5)
            edge = {
                "id": eid,
                "source": e["source_jrn"],
                "target": e["target_jrn"],
                "animated": conf < 0.5,
                "data": {
                    "tier": e.get("tier", 3),
                    "confidence": conf,
                    "queryType": e.get("query_type", ""),
                    "provenance": e.get("provenance", []),
                    "corroboratedBy": e.get("corroborated_by", []),
                    "firstSeen": e.get("first_seen"),
                    "lastConfirmed": e.get("last_confirmed"),
                },
                "style": self._edge_style(conf),
            }
            if column_level:
                edge["sourceHandle"] = e.get("source_column", "")
                edge["targetHandle"] = e.get("target_column", "")
            edges.append(edge)
        return edges

    @staticmethod
    def _edge_style(confidence: float) -> dict:
        if confidence >= 0.9:
            return {"stroke": "#16a34a", "strokeWidth": 2}
        if confidence >= 0.5:
            return {"stroke": "#ca8a04", "strokeWidth": 1.5}
        return {"stroke": "#dc2626", "strokeWidth": 1, "strokeDasharray": "5,5"}

    @staticmethod
    def _count_by_tier(edges: list[dict]) -> dict[str, int]:
        counts: dict[str, int] = {}
        for e in edges:
            t = str(e.get("tier", 3))
            counts[t] = counts.get(t, 0) + 1
        return counts
```

### 5.2 Impact Analysis Service

```python
class ImpactAnalysisService:
    """BFS downstream traversal — find all assets affected if a table changes."""

    def __init__(self, db: Database) -> None:
        self._db = db

    async def analyze(
        self,
        start_jrn: str,
        depth: int = 5,
        min_confidence: float = 0.5,
    ) -> dict:
        visited: dict[str, dict] = {}
        queue: deque[tuple[str, int, list[str]]] = deque(
            [(start_jrn, 0, [start_jrn])]
        )

        while queue:
            current, hop, path = queue.popleft()
            if current in visited or hop >= depth:
                continue
            visited[current] = {"distance": hop, "path": path}

            for edge in self._db.lineage_events.find({
                "source_jrn": current,
                "is_column_level": False,
                "confidence": {"$gte": min_confidence},
            }):
                target = edge["target_jrn"]
                if target not in visited:
                    queue.append((target, hop + 1, path + [target]))

        # Enrich with metadata
        impacted = []
        for jrn, info in visited.items():
            if jrn == start_jrn:
                continue
            dist = self._db.data_distributions.find_one({"jrn": jrn})
            impacted.append({
                "jrn": jrn,
                "name": dist["name"] if dist else jrn.split("/")[-1],
                "platform": dist["platform"] if dist else "UNKNOWN",
                "distance": info["distance"],
                "path": info["path"],
                "impact_type": "direct" if info["distance"] == 1 else "indirect",
            })

        impacted.sort(key=lambda x: x["distance"])
        return {
            "root_jrn": start_jrn,
            "impacted_assets": impacted,
            "total_impacted": len(impacted),
            "max_distance": max((i["distance"] for i in impacted), default=0),
        }
```

---

## 6. MongoDB Indexes Required

Add these indexes to support the API queries efficiently:

```python
# Asset catalog search
db.data_distributions.create_index([("name", "text")])
db.data_distributions.create_index([("platform", 1), ("source.sourceStatus", 1)])
db.data_distributions.create_index([("source.sourceTypeName", 1)])
db.data_distributions.create_index([("environment", 1)])

# Asset detail
db.data_distributions.create_index([("jrn", 1)], unique=True)

# Attributes
db.data_attributes.create_index([("parent_jrn", 1), ("ordinal", 1)])

# Lineage graph traversal
db.lineage_events.create_index([("source_jrn", 1), ("is_column_level", 1), ("confidence", -1)])
db.lineage_events.create_index([("target_jrn", 1), ("is_column_level", 1), ("confidence", -1)])
db.lineage_events.create_index([("tier", 1), ("confidence", -1)])
```

---

## 7. React Component Architecture

```
src/
  modules/
    assets/
      pages/
        AssetCatalogPage.tsx            — search + browse
        AssetDetailPage.tsx             — tabbed detail view
        ImpactAnalysisPage.tsx          — downstream impact view
      components/
        AssetTable.tsx                  — paginated asset list
        AssetFilters.tsx                — platform/type/status/env facets
        AssetSearch.tsx                 — search input with debounce
        AssetOverview.tsx               — metadata display
        AttributeTable.tsx              — column list with lineage indicator
        LineageGraph.tsx                — React Flow wrapper
        LineageControls.tsx             — direction/depth/confidence/view controls
        EdgeDetailPanel.tsx             — selected edge detail sidebar
        ColumnTableNode.tsx             — custom React Flow node with handles
        TableNode.tsx                   — simple table-level node
        ImpactTable.tsx                 — impact analysis results
        PlatformBadge.tsx               — platform icon/color badge
        ConfidenceBadge.tsx             — tier/confidence visual indicator
        LineageLegend.tsx               — confidence color legend
      hooks/
        useAssets.ts                    — SWR/TanStack Query for asset list
        useAssetDetail.ts               — SWR for single asset
        useAttributes.ts                — SWR for attribute list
        useLineageGraph.ts              — SWR for lineage graph
        useImpactAnalysis.ts            — SWR for impact data
      api/
        assetApi.ts                     — API client functions
      types/
        asset.ts                        — TypeScript types
        lineage.ts                      — graph/edge/node types
      utils/
        layout.ts                       — dagre layout helper
        edgeStyle.ts                    — confidence → style mapping
```

### 7.1 Key React Components

#### LineageGraph.tsx — The Core Visualization

```tsx
import { useCallback, useEffect, useMemo } from 'react';
import ReactFlow, {
  Background, Controls, MiniMap, Panel,
  useNodesState, useEdgesState,
} from 'reactflow';
import dagre from '@dagrejs/dagre';
import { useLineageGraph } from '../hooks/useLineageGraph';
import { TableNode } from './TableNode';
import { ColumnTableNode } from './ColumnTableNode';
import { LineageControls } from './LineageControls';
import { EdgeDetailPanel } from './EdgeDetailPanel';
import { LineageLegend } from './LineageLegend';
import 'reactflow/dist/style.css';

const nodeTypes = {
  tableNode: TableNode,
  columnTableNode: ColumnTableNode,
};

interface LineageGraphProps {
  jrn: string;
  defaultDepth?: number;
}

export function LineageGraph({ jrn, defaultDepth = 2 }: LineageGraphProps) {
  const [params, setParams] = useState({
    depth: defaultDepth,
    direction: 'both',
    minConfidence: 0,
    columnLevel: false,
  });
  const [selectedEdge, setSelectedEdge] = useState(null);
  const { data, isLoading } = useLineageGraph(jrn, params);
  const [nodes, setNodes, onNodesChange] = useNodesState([]);
  const [edges, setEdges, onEdgesChange] = useEdgesState([]);

  useEffect(() => {
    if (!data) return;
    const laidOut = applyDagreLayout(
      data.nodes,
      [...data.edges, ...(data.column_edges || [])],
    );
    setNodes(laidOut);
    setEdges([...data.edges, ...(data.column_edges || [])]);
  }, [data]);

  const onEdgeClick = useCallback((_, edge) => {
    setSelectedEdge(edge.data);
  }, []);

  if (isLoading) return <div>Loading lineage...</div>;

  return (
    <div style={{ width: '100%', height: '600px' }}>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        nodeTypes={nodeTypes}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        onEdgeClick={onEdgeClick}
        fitView
      >
        <Background />
        <Controls />
        <MiniMap />
        <Panel position="top-left">
          <LineageControls
            params={params}
            onChange={setParams}
            stats={data?.stats}
          />
        </Panel>
        <Panel position="bottom-left">
          <LineageLegend />
        </Panel>
      </ReactFlow>
      {selectedEdge && (
        <EdgeDetailPanel
          edge={selectedEdge}
          onClose={() => setSelectedEdge(null)}
        />
      )}
    </div>
  );
}
```

#### ColumnTableNode.tsx — Column-Level Node with Handles

```tsx
import { Handle, Position } from 'reactflow';
import { PlatformBadge } from './PlatformBadge';

export function ColumnTableNode({ data }) {
  return (
    <div className={`lineage-node ${data.isRoot ? 'root' : ''}`}>
      <div className="node-header">
        <PlatformBadge platform={data.platform} />
        <span className="node-title">{data.label}</span>
      </div>
      <div className="node-schema">{data.schema}</div>
      <div className="column-list">
        {data.columns?.map((col) => (
          <div key={col.name} className="column-row">
            <Handle
              type="target"
              position={Position.Left}
              id={col.name}
              className="column-handle"
            />
            <span className="col-name">{col.name}</span>
            <span className="col-type">{col.type}</span>
            <Handle
              type="source"
              position={Position.Right}
              id={col.name}
              className="column-handle"
            />
          </div>
        ))}
      </div>
    </div>
  );
}
```

#### layout.ts — Dagre Layout Helper

```typescript
import dagre from '@dagrejs/dagre';
import type { Node, Edge } from 'reactflow';

const NODE_WIDTH = 220;
const NODE_HEIGHT_BASE = 60;
const COLUMN_ROW_HEIGHT = 24;

export function applyDagreLayout(nodes: Node[], edges: Edge[]): Node[] {
  const g = new dagre.graphlib.Graph();
  g.setGraph({
    rankdir: 'LR',
    ranksep: 120,
    nodesep: 40,
    edgesep: 20,
  });
  g.setDefaultEdgeLabel(() => ({}));

  nodes.forEach((n) => {
    const columnCount = n.data.columns?.length || 0;
    const height = columnCount > 0
      ? NODE_HEIGHT_BASE + columnCount * COLUMN_ROW_HEIGHT
      : NODE_HEIGHT_BASE;
    g.setNode(n.id, { width: NODE_WIDTH, height });
  });

  edges.forEach((e) => g.setEdge(e.source, e.target));
  dagre.layout(g);

  return nodes.map((n) => {
    const pos = g.node(n.id);
    return {
      ...n,
      position: {
        x: pos.x - NODE_WIDTH / 2,
        y: pos.y - (g.node(n.id).height || NODE_HEIGHT_BASE) / 2,
      },
    };
  });
}
```

---

## 8. Cross-Platform Lineage

The graph naturally handles cross-platform lineage because edges are keyed by JRN, not by platform. A Databricks table feeding a Tableau datasource is just an edge where `source_jrn` contains `databricks/...` and `target_jrn` contains `tableau/...`. The UI renders different platform badges on the nodes:

```
┌───────────────┐         ┌───────────────┐         ┌───────────────┐
│ trades        │         │ positions     │         │ risk_dash     │
│ ⬡ DATABRICKS  │────────▶│ ⬡ DATABRICKS  │────────▶│ ⬡ TABLEAU     │
│ PROD          │  T1 1.0 │ PROD          │  T2 0.9 │ PROD          │
└───────────────┘         └───────────────┘         └───────────────┘
```

The OpenLineage receiver (S10a) already normalizes Alteryx-to-Databricks and Alteryx-to-Snowflake edges. The graph assembly just traverses whatever edges exist, regardless of platform boundaries.

---

## 9. Performance Considerations

### 9.1 Query Optimization

The BFS traversal issues 2 MongoDB queries per hop per node (upstream + downstream). For depth=2 and 20 nodes, that's ~80 queries. Each hits indexed fields (`source_jrn` or `target_jrn` + `is_column_level` + `confidence`).

For large graphs (>100 nodes), consider:
- Server-side caching with TTL (graph unlikely to change minute-to-minute)
- Cursor-based pagination on edges (cap at 500 edges per response)
- Separate column edge loading (load table graph first, then column edges on demand)

### 9.2 Frontend Performance

React Flow handles up to ~500 nodes smoothly. Beyond that:
- Virtual rendering (only render visible nodes)
- Progressive loading (load depth=1 first, expand on click)
- Collapse distant nodes into summary badges ("+ 15 more downstream")

---

## 10. Platform Badge Colors

Consistent visual identity across the SPA:

| Platform | Color | Icon |
|---|---|---|
| DATABRICKS | `#FF3621` (red-orange) | Databricks logo |
| SNOWFLAKE | `#29B5E8` (sky blue) | Snowflake logo |
| TABLEAU | `#E97627` (orange) | Tableau logo |
| ORACLE | `#F80000` (red) | Oracle logo |
| ALTERYX | `#0078C0` (blue) | Alteryx logo |
