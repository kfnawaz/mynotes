# Databricks Unity Catalog — Extraction Query Plan

**Project:** In-House Replacement of Atlan Crawlers/Miners
**Purpose:** Defines exactly how the new crawler and miner extract metadata and lineage from Databricks Unity Catalog — the query plan feeding the `DataDistribution` and `DataAttribute` collections (Workstream 1) and the lineage entities (Workstream 2).
**Companion artifacts:** `atlan_replacement_proposal.md`, `mvp_parity_matrix.md`, `databricks_currentstate_inventory_template.md`, `data_compass_model.md`.

> **Verification note:** Unity Catalog `information_schema` and `system.*` table/column names should be confirmed against the live estate during Phase 1. Where this plan names a specific table or column, treat it as the design intent to verify, not a guarantee. Mark deviations during inventory.

---

## 1. Extraction Surfaces — Overview

Databricks Unity Catalog exposes metadata through two complementary surfaces. The plan uses both.

| Surface | Scope | Used for | Notes |
|---|---|---|---|
| **`information_schema`** | Per-catalog | Crawler — tables, views, columns, schemas | One `information_schema` per catalog; query each in-scope catalog |
| **`system` catalog** | Account-wide | Miner — lineage; cross-catalog object inventory | Single `system` catalog per metastore; `system.access.*`, `system.information_schema.*` |

**Design choice:** Use `system.information_schema.*` (account-wide) for the crawler where possible — it avoids iterating per-catalog and gives one consistent query surface. Fall back to per-catalog `information_schema` only if account-wide system tables are not enabled or lack a needed column. Confirm `system` schema availability in Phase 1.

**Access method:** All queries run as SQL against an owned SQL warehouse (the project already owns warehouses). The crawler connects via the Databricks SQL connector / JDBC, executes the queries below, and streams results into the normalization layer.

---

## 2. Crawler Query Plan — `DataDistribution` (Tables & Views)

Target collection: `DataDistribution`. Maps to model entity `Distribution`. Acceptance attributes per `mvp_parity_matrix.md` §3.

### 2.1 Primary Query — Table & View Inventory

Source: `system.information_schema.tables` (account-wide) or per-catalog `information_schema.tables`.

Columns to select:

| UC column | Feeds Catalog DB attribute | Transform |
|---|---|---|
| `table_catalog` | `catalogName` | direct |
| `table_schema` | `databaseName` | direct |
| `table_name` | `name` | direct |
| `table_type` | `source.sourceTypeName` | `MANAGED`/`EXTERNAL` → `TABLE`; `VIEW` → `VIEW` (confirm value set) |
| `comment` | `description` | direct; may be null |
| `created` | `source.sourceCreateTime` | timestamp normalization |
| `last_altered` | `source.sourceUpdateTime` | timestamp normalization |
| `table_owner` | (candidate for `owners[]`) | confirm against model owner handling |

Filter: restrict to in-scope catalogs/schemas per the Section A.3 scope table from the inventory. Exclude `information_schema` and `system` catalogs themselves.

### 2.2 Derived & Crawler-Owned Attributes

Not from the table query — assembled by the normalization layer:

| Attribute | Source |
|---|---|
| `source.sourceQualifiedName` | Constructed: `table_catalog.table_schema.table_name` |
| `source.sourceGuid` | UC object identity — see §2.4 |
| `source.sourceStatus` | Derived from delta logic (§5): present = `ACTIVE`, absent = `DELETED` |
| `source.sourceSystem` | Constant `DATABRICKS` |
| `source.syncStatus`, `source.syncTime` | Crawler-owned (set at write time) |
| `displayName` | = `name` |
| `jrn` | JRN minting service call |
| `environment` | Workspace-to-environment lookup |
| `connectionName` | Databricks workspace name |

### 2.3 Supplementary Queries (extended Workstream 1 coverage)

Beyond MVP, for `Dataset` / `DataModelEntity` / `DataService` enrichment:

| Need | Source | Notes |
|---|---|---|
| Schema inventory | `system.information_schema.schemata` | catalog/schema hierarchy, comments |
| Catalog inventory | `system.information_schema.catalogs` | top-level catalog list |
| View definitions | `information_schema.views` (`view_definition`) | the SQL text of views — also a miner input |
| Table constraints | `information_schema.table_constraints`, `key_column_usage` | primary/foreign keys |
| Storage/format detail | `DESCRIBE EXTENDED` or `system.information_schema.tables` extended columns | for `Distribution.format[]`, physical spec |

### 2.4 Object Identity (`source.sourceGuid`)

Unity Catalog does not expose a single universal GUID column across all `information_schema` views the way Atlan assigns one. Options to confirm in Phase 1:

- A stable identity column on the system tables (e.g. a `table_id` if present in the live estate).
- Otherwise, treat the fully-qualified name `catalog.schema.object` as the stable natural identity and let the JRN minting service own canonical identity.

**Resolve during inventory.** This affects how the delta logic (§5) detects "same object."

---

## 3. Crawler Query Plan — `DataAttribute` (Columns)

Target collection: `DataAttribute`. Maps to model entity `DataModelAttribute`. Acceptance attributes per `mvp_parity_matrix.md` §4.

### 3.1 Primary Query — Column Inventory

Source: `system.information_schema.columns` (account-wide) or per-catalog `information_schema.columns`.

| UC column | Feeds Catalog DB attribute | Transform |
|---|---|---|
| `table_catalog` | `catalogName` | direct |
| `table_schema` | `databaseName` | direct |
| `table_name` | `tableName` | direct — links column → parent `DataDistribution` |
| `column_name` | `name` | direct |
| `full_data_type` (or `data_type`) | `dataType` | **normalize → model enum** (§3.3) |
| `comment` | `description` | direct; may be null |
| `ordinal_position` | (candidate for attribute `position`) | extended coverage |
| `is_nullable` | (candidate for `isNullable`) | extended coverage |
| `column_default` | (candidate for extended attribute) | extended coverage |

Filter: same in-scope catalog/schema restriction as §2.1.

### 3.2 Derived & Crawler-Owned Attributes

| Attribute | Source |
|---|---|
| `source.sourceQualifiedName` | Constructed: `catalog.schema.table.column` |
| `source.sourceGuid` | UC column identity — same resolution as §2.4 |
| `source.sourceTypeName` | Constant `COLUMN` |
| `source.sourceStatus` | Delta logic (§5) |
| `source.sourceSystem` | Constant `DATABRICKS` |
| `source.syncStatus`, `source.syncTime` | Crawler-owned |
| `displayName` | = `name` |
| `jrn` | JRN minting service — must reference parent table's `jrn` |
| `environment` | Workspace-to-environment lookup |
| `connectionName` | Workspace name |

### 3.3 `dataType` Normalization

Raw UC type → model data-type enum (`BOOLEAN, CHAR, STRING, DATE, TIME, TIMESTAMP, INTEGER, LONG, DOUBLE, FLOAT, DECIMAL, UUID, JSON, XML, BYTE, BINARY, ARRAY, OTHER`).

Indicative mapping (complete and verify against the actual UC types in the estate during Phase 1):

| Raw UC type | Model enum |
|---|---|
| `boolean` | `BOOLEAN` |
| `string`, `varchar` | `STRING` |
| `char` | `CHAR` |
| `int`, `integer`, `smallint`, `tinyint` | `INTEGER` |
| `bigint`, `long` | `LONG` |
| `float`, `real` | `FLOAT` |
| `double` | `DOUBLE` |
| `decimal`, `numeric` | `DECIMAL` |
| `date` | `DATE` |
| `timestamp`, `timestamp_ntz` | `TIMESTAMP` |
| `binary` | `BINARY` |
| `array<...>` | `ARRAY` |
| `struct<...>`, `map<...>`, `variant` | confirm — likely `JSON` or `OTHER` |
| *(anything unmatched)* | `OTHER` |

Complex types (`struct`, `map`, `array<struct>`) need an explicit decision: collapse to `ARRAY`/`OTHER`, or flatten nested fields into child attributes. Decide in Phase 2 design.

---

## 4. Miner Query Plan — Lineage (Workstream 2)

Target entities: `DataFlow`, `DataProcess`, and the event entities. Source: the `system.access.*` lineage tables.

### 4.1 Lineage Source Tables

| System table | Provides | Feeds |
|---|---|---|
| `system.access.table_lineage` | Table-to-table lineage events | `DataFlow` (table-level), `DataProcess` |
| `system.access.column_lineage` | Column-to-column lineage events | `DataFlow` (column-level precision) |

Both are event-style tables — each row is a lineage observation tied to a query/job execution with a timestamp. The miner aggregates these into the catalog's lineage entities.

### 4.2 Key Columns (confirm against live schema)

`table_lineage` typically exposes: source table (catalog/schema/name), target table, the entity type that produced the event (notebook, job, query, pipeline), and an event timestamp. `column_lineage` adds source/target column names.

| Lineage attribute | Feeds Catalog DB |
|---|---|
| Source object FQN | `DataFlow.dataProvider` / `DataProcess` inbound |
| Target object FQN | `DataFlow.dataConsumer` / `DataProcess` outbound |
| Event timestamp | `DataFlow.lastObservedTimestamp`, event `eventTimestamp` |
| Producing entity (job/notebook/query) | `DataProcess`, `node` references |
| Column source/target | `DataFlow` column-level precision |

### 4.3 Miner Processing Logic

1. **Read lineage events** incrementally (by event timestamp watermark — see §5).
2. **Resolve object FQNs** to existing `DataDistribution` / `DataAttribute` `jrn`s via the crawler-populated catalog. Lineage referencing an un-crawled object is a flag — crawler and miner must cover the same scope.
3. **Aggregate** repeated edges into `DataFlow` records; set `precision` (`TABLE` vs `COLUMN`) from which source table the edge came from.
4. **Derive `DataProcess`** from the producing entity (job/notebook), capturing transformations where determinable.
5. **Emit events** (`DataConsumptionEvent`, `DataProductionEvent`, etc.) per the model.

### 4.4 SQL Parsing

Where lineage must be derived from query text (`information_schema.views.view_definition`, or query history beyond what `system.access.*` resolves), use a proven SQL parser (`sqlglot` / `sqllineage`) — not a hand-written parser. Scope CTE, subquery, and dialect handling explicitly in Phase 2.

### 4.5 Miner Permissions

The crawler service principal needs `SELECT` on `system.access.table_lineage` and `system.access.column_lineage`. Confirm `system.access` schema is enabled on the metastore (it is not always on by default) — flag in Phase 1 inventory Section D.

---

## 5. Incremental Crawl & Delta Strategy

The Atlan sync is delta-based (`processDeltaFromAtlan()`); the new crawler must match. Full scans do not scale to the Databricks estate volume.

### 5.1 Change Detection — Crawler

| Approach | Mechanism | Trade-off |
|---|---|---|
| **Timestamp watermark** | Compare `last_altered` against the last successful crawl watermark | Simple; misses deletes (a dropped table has no row to compare) |
| **Full inventory diff** | Pull current FQN set, diff against catalog's known set | Catches deletes; heavier query |

**Recommended hybrid:** every run, pull the **full lightweight FQN inventory** (catalog/schema/name only — cheap) to detect creates and deletes; pull **full attribute detail only for objects whose `last_altered` exceeds the watermark**, plus newly-seen objects. This catches all three change classes (create/update/delete) without full attribute extraction every run.

### 5.2 Soft-Delete

An FQN in the catalog but absent from the current inventory pull → set `source.sourceStatus = DELETED`. **Never hard-delete.** Columns of a deleted table are likewise marked `DELETED`.

### 5.3 Change Detection — Miner

Lineage tables are event tables — use an **event-timestamp watermark**: read only lineage events newer than the last processed timestamp. Store the watermark per run.

### 5.4 History Emission

On any change to a `DataDistribution` / `DataAttribute` document, emit the corresponding `*History` document (parent attributes + historical timestamp). This responsibility moves from the Atlan sync service to the new crawler's load layer.

### 5.5 Raw Payload Staging

Stage raw query results before normalization. This allows re-running normalization (e.g. after a `dataType` mapping fix) without re-querying Databricks.

---

## 6. Execution Model

### 6.1 Crawl Sequence (per run)

1. Resolve in-scope catalogs/schemas (from config / inventory Section A.3).
2. Lightweight FQN inventory query (tables + views) → delta computation (§5.1).
3. Detail query for changed/new tables & views → stage raw.
4. Lightweight column FQN inventory → delta.
5. Detail query for changed/new columns → stage raw.
6. Normalize staged payloads → call JRN minting service, apply `dataType` enum + environment lookup.
7. Load to Catalog DB: dual-write `source.*` + `atlan.*`, upsert, soft-delete, emit `*History`.
8. Miner: read `system.access.*` since watermark → resolve FQNs → aggregate → load lineage entities.
9. Record watermarks; emit run metrics.

### 6.2 Orchestration

A scheduler (e.g. Airflow) replaces the VSI cron jobs. Each platform and each major step is an idempotent, independently re-runnable task. One workspace's or one catalog's failure must not block others.

### 6.3 Warehouse & Concurrency

Queries run on an owned SQL warehouse. For large estates, parallelize the detail queries by catalog or schema. Size warehouse and concurrency against the volume figures from inventory Section A.4. Be mindful of `information_schema` query cost on very large catalogs — partition by schema if needed.

### 6.4 Observability

Per run: object counts (created/updated/deleted) per catalog, query durations, normalization failures, lineage events processed, watermark values. A recurring reconciliation job diffs crawler output against expected catalog state.

---

## 7. Permissions Summary

The new crawler's Databricks service principal requires:

| Permission | On | For |
|---|---|---|
| `USE CATALOG` | each in-scope catalog | access |
| `USE SCHEMA` | each in-scope schema | access |
| `SELECT` | `information_schema` views / `system.information_schema.*` | crawler metadata queries |
| `SELECT` | `system.access.table_lineage`, `system.access.column_lineage` | miner lineage queries |
| SQL warehouse usage | the owned crawl warehouse | query execution |

Dedicated to the crawler — do **not** reuse Atlan's service principal. Provision during Phase 1 (inventory Section F).

---

## 8. Open Items for Phase 1 / Phase 2

1. **Verify `system` catalog availability** — confirm `system.information_schema.*` and `system.access.*` are enabled on the metastore; some require explicit enablement.
2. **Object identity** — confirm whether a stable `table_id`/identity column exists, or whether FQN + JRN minting service is the identity basis (§2.4).
3. **`table_type` value set** — confirm the exact values UC returns so the `TABLE`/`VIEW` mapping is correct (§2.1).
4. **`dataType` raw values** — enumerate the actual UC types present in the estate to complete the normalization table (§3.3).
5. **Complex types** — decide handling for `struct`/`map`/`variant` columns (collapse vs. flatten) (§3.3).
6. **`owners[]`** — confirm whether `table_owner` from UC feeds the model's `owners[]`/`owners[].sid`, or whether ownership comes from elsewhere.
7. **Lineage table schema** — verify exact column names of `system.access.table_lineage` / `column_lineage` against the live estate (§4.2).
8. **JRN minting** — parent/child reference and idempotency (carried open from the proposal §7.3).
9. **`information_schema` query cost** — assess performance on the largest catalogs; decide schema-level partitioning.
