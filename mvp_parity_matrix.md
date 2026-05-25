# MVP Parity Matrix — Atlan Crawler Replacement (Databricks)

**Project:** In-house Replacement of Atlan Crawlers/Miners
**Purpose:** Defines the minimum viable acceptance criteria for the new Databricks crawler. The new crawler reaches MVP parity when it reproduces every attribute below, sourced from Databricks Unity Catalog instead of Atlan.
**Basis:** `atlansynctocatalogdb.docx` — the realized Atlan→catalog sync contract.
**Catalog DB:** MongoDB.

---

## 1. Scope Decision

The Atlan sync writes **4 MongoDB collections** plus audit-history twins. Only **2 are crawler-sourced** and therefore in MVP scope.

| Collection | Atlan source asset | In MVP crawler scope? | Rationale |
|---|---|---|---|
| `DataDistribution` | Atlan Table/View assets | **YES** | Crawled from Databricks tables/views |
| `DataAttribute` | Atlan Column assets | **YES** | Crawled from Databricks columns |
| `DataDomain` | Atlan DataDomain assets | No | Created in Atlan UI, not crawled from a platform |
| `DataProduct` | Atlan DataProduct assets | No | Created in Atlan UI, not crawled from a platform |
| `*History` collections | Audit twins | Indirect | Same attributes + historical timestamp; satisfied once the parent collection's writer emits change events |

**MVP crawler deliverable:** populate `DataDistribution` and `DataAttribute` for Databricks, replacing `DataDistributionSyncFromAtlanService.processDeltaFromAtlan()` and `DataAttributeSyncFromAtlanService.processDeltaFromAtlan()`.

> `DataDomain` / `DataProduct` sourcing post-Atlan is a separate decision (new UI, or another process) — out of crawler scope but must be resolved before full Atlan decommission.

---

## 2. Provenance Block — `source.*` (DECIDED: Option B)

Every collection currently embeds an `atlan.*` sub-document. **Decision: rename to a neutral, platform-agnostic `source.*` block.** The crawler writes `source.*`; all downstream consumers (sync code, internal UI, MongoDB indexes, queries, reports) must be migrated off `atlan.*`.

| Current `atlan.*` field | New `source.*` field | Databricks Unity Catalog source |
|---|---|---|
| `atlan.guid` | `source.sourceGuid` | UC object identity (catalog.schema.table identity / table_id) |
| `atlan.qualifiedName` | `source.sourceQualifiedName` | UC three-level name `catalog.schema.object` |
| `atlan.typeName` | `source.sourceTypeName` | `TABLE` / `VIEW` from `information_schema.tables` |
| `atlan.status` | `source.sourceStatus` | Derived: present at source = `ACTIVE`; absent = `DELETED` |
| `atlan.createTime` | `source.sourceCreateTime` | `information_schema.tables.created` |
| `atlan.updateTime` | `source.sourceUpdateTime` | `information_schema.tables.last_altered` |
| `atlan.syncStatus` | `source.syncStatus` | Crawler-owned: `PROCESSED`, `IN_PROGRESS`, etc. |
| `atlan.syncTime` | `source.syncTime` | Crawler-owned: crawl write timestamp |
| — | `source.sourceSystem` | New constant: `DATABRICKS` |

**Migration dependency:** The `source.*` rename is a catalog-DB schema-contract change affecting the UI team and anyone querying the collections. The crawler write layer targets `source.*` from day one. Coordinate the cutover of `atlan.*` readers with the UI team. If a phased approach is needed, the crawler could dual-write `atlan.*` + `source.*` during a transition window, then drop `atlan.*` once all readers are migrated — confirm whether dual-write is wanted or a clean switch.

---

## 3. DataDistribution — Parity Checklist

Source: Atlan Table/View assets (data ports). Target collection: `DataDistribution`. Maps to data-dictionary entity `Distribution`.

| # | Catalog attribute | Meaning | Databricks UC source | Notes |
|---|---|---|---|---|
| 1 | `_id` | MongoDB document ID | n/a — MongoDB-assigned | Not crawler-produced |
| 2 | `source.sourceGuid` | Source object GUID (was `atlan.guid`) | UC table identity | See §2 |
| 3 | `source.sourceQualifiedName` | Qualified name (was `atlan.qualifiedName`) | `catalog.schema.object` | See §2 |
| 4 | `source.sourceStatus` | Status ACTIVE/DELETED (was `atlan.status`) | Derived from presence at source | See §2; drives soft-delete |
| 5 | `source.sourceTypeName` | TABLE or VIEW (was `atlan.typeName`) | `information_schema.tables.table_type` | See §2 |
| 6 | `source.syncStatus` | Sync status (was `atlan.syncStatus`) | Crawler-owned | See §2 |
| 7 | `source.syncTime` | Sync timestamp (was `atlan.syncTime`) | Crawler-owned | See §2 |
| 8 | `source.sourceSystem` | Source platform constant | Constant `DATABRICKS` | New field, see §2 |
| 9 | `name` | Table/View name | `information_schema.tables.table_name` | |
| 10 | `displayName` | Display name | Derived from `name` unless overridden | Confirm derivation rule |
| 11 | `description` | Description | `information_schema.tables.comment` | |
| 12 | `jrn` | Unique identifier (JPMorgan Resource Name) | **JRN minting service** | Crawler calls the existing JRN minting service — does not construct keys itself. See §5 |
| 13 | `environment` | Environment name | **Workspace-to-environment lookup table** → `PROD` / `UAT` / `DEV` | Crawler resolves the Databricks workspace against a maintained lookup. See §5 |
| 14 | `connectionName` | Database connection name | **Databricks workspace name** | Direct: `connectionName` = the workspace the object was crawled from |
| 15 | `databaseName` | Database/Schema name | UC schema name | |
| 16 | `catalogName` | Catalog name (for Unity Catalog) | UC catalog name | UC-only; null/different for Hive metastore |

---

## 4. DataAttribute — Parity Checklist

Source: Atlan Column assets. Target collection: `DataAttribute`. Maps to data-dictionary entity `DataModelAttribute` / `Distribution.physicalSpecification.dataAttributes[]`.

| # | Catalog attribute | Meaning | Databricks UC source | Notes |
|---|---|---|---|---|
| 1 | `_id` | MongoDB document ID | n/a — MongoDB-assigned | Not crawler-produced |
| 2 | `source.sourceGuid` | Source column GUID (was `atlan.guid`) | UC column identity | See §2 |
| 3 | `source.sourceQualifiedName` | Qualified name (was `atlan.qualifiedName`) | `catalog.schema.table.column` | See §2 |
| 4 | `source.sourceStatus` | Status (was `atlan.status`) | Derived from presence at source | See §2 |
| 5 | `source.sourceTypeName` | COLUMN (was `atlan.typeName`) | Constant `COLUMN` | See §2 |
| 6 | `source.syncStatus` | Sync status (was `atlan.syncStatus`) | Crawler-owned | See §2 |
| 7 | `source.syncTime` | Sync timestamp (was `atlan.syncTime`) | Crawler-owned | See §2 |
| 8 | `source.sourceSystem` | Source platform constant | Constant `DATABRICKS` | New field, see §2 |
| 9 | `name` | Column/Attribute name | `information_schema.columns.column_name` | |
| 10 | `displayName` | Display name | Derived from `name` unless overridden | Confirm derivation rule |
| 11 | `description` | Description | `information_schema.columns.comment` | |
| 12 | `dataType` | Data type | `information_schema.columns.data_type` → **normalized to model data-type enum** | `BOOLEAN, CHAR, STRING, DATE, TIME, TIMESTAMP, INTEGER, LONG, DOUBLE, FLOAT, DECIMAL, UUID, JSON, XML, BYTE, BINARY, ARRAY, OTHER`. Crawler maps raw UC type → enum |
| 13 | `jrn` | Unique identifier (JPMorgan Resource Name) | **JRN minting service** | Crawler calls the minting service. Must reference parent table's `jrn`. See §5 |
| 14 | `environment` | Environment name | **Workspace-to-environment lookup table** → `PROD` / `UAT` / `DEV` | Same rule as Distribution |
| 15 | `connectionName` | Connection name | **Databricks workspace name** | Same rule as Distribution |
| 16 | `databaseName` | Database/Schema name | UC schema name | |
| 17 | `tableName` | Parent table name | `information_schema.columns.table_name` | Links column → parent Distribution |
| 18 | `catalogName` | Catalog name | UC catalog name | |

---

## 5. Crawler-Derived Values — Resolved Rules

These attributes are not read directly from Databricks; the crawler derives them. All rules below are now settled.

### 5.1 `jrn` — JRN Minting Service
`jrn` is the JPMorgan Resource Name, the universal join key across the catalog model. **The crawler calls the existing JRN minting service** — it does not construct keys itself. This removes the deterministic-key design risk previously flagged.

Build notes:
- Integrate the minting service into the crawler write path for both `DataDistribution` and `DataAttribute`.
- Confirm the minting service is idempotent — re-crawling the same UC object must yield the same `jrn`, or parent/child links and history break.
- Confirm how `DataAttribute` `jrn` references its parent `DataDistribution` `jrn` (whether the minting service handles the hierarchy or the crawler passes the parent `jrn` as input).
- Confirm rename behavior at source: does a renamed UC object mint a new `jrn` or retain the existing one?

### 5.2 `environment` — Workspace-to-Environment Lookup
`environment` is `PROD` / `UAT` / `DEV`, resolved from a **workspace-to-environment lookup table**. The crawler maps the Databricks workspace an object was crawled from against this lookup.

Build notes:
- The crawler carries (or queries) a maintained `workspace → environment` mapping.
- Confirm where this lookup lives (config file, catalog DB collection, separate service) and who owns keeping it current as workspaces are onboarded.
- A workspace missing from the lookup must fail loudly, not default silently.

### 5.3 `connectionName` — Workspace Name
`connectionName` is the **Databricks workspace name** — a direct value, no lookup. The crawler stamps each `DataDistribution` and `DataAttribute` document with the name of the workspace it was crawled from.

### 5.4 `dataType` — Normalized to Model Enum
`DataAttribute.dataType` is **normalized** from the raw Unity Catalog type to the model's data-type enum (`BOOLEAN, CHAR, STRING, DATE, TIME, TIMESTAMP, INTEGER, LONG, DOUBLE, FLOAT, DECIMAL, UUID, JSON, XML, BYTE, BINARY, ARRAY, OTHER`).

Build notes:
- The crawler needs a UC-type → enum mapping table.
- Define the fallback: any unrecognized UC type maps to `OTHER`.
- Confirm the mapping table is reviewed when Databricks introduces new types.

---

## 6. Delta / Soft-Delete Behavior

The Atlan sync uses `processDeltaFromAtlan()` — i.e. incremental, not full reload. The new crawler must replicate:
- **Incremental detection:** identify created/updated/deleted UC objects since last crawl.
- **Soft-delete:** `source.sourceStatus = DELETED` is how removal is represented — there is no hard delete. The crawler must set status to `DELETED` (not remove the document) when a UC object disappears.
- **History emission:** each change must produce a `*History` document (parent attributes + historical timestamp).

---

## 7. Open Questions to Confirm Before Build

**Resolved:**
- ~~Provenance block~~ → **Decided: Option B**, rename to `source.*` (§2).
- ~~`jrn` format~~ → **Resolved: existing JRN minting service** is called by the crawler (§5.1).
- ~~`environment` derivation~~ → **Resolved: workspace-to-environment lookup table** (§5.2).
- ~~`connectionName` derivation~~ → **Resolved: Databricks workspace name** (§5.3).
- ~~`dataType` handling~~ → **Resolved: normalized to model enum** (§5.4).

**Still open:**
1. **`source.*` cutover** — clean switch, or crawler dual-writes `atlan.*` + `source.*` during a transition window? Coordinate with the UI team (§2).
2. **JRN minting service** — idempotency confirmation; how `DataAttribute` `jrn` references its parent; rename behavior (§5.1).
3. **Workspace-to-environment lookup** — where it lives, who owns it, behavior for an unmapped workspace (§5.2).
4. **`displayName` derivation** — derived from `name`, or a separate source value? (Applies to both collections.)
5. **UC-type → enum mapping table** — ownership and review cadence; `OTHER` fallback confirmed (§5.4).
6. **Hive metastore workspaces** — `catalogName` is "for Unity Catalog"; confirm behavior for any non-UC workspaces (different code path).
7. **History collections** — confirm `DataDistributionHistory` / `DataAttributeHistory` exist and their exact trigger semantics.

---

## 8. MVP Acceptance Definition

The new Databricks crawler passes MVP parity when, run in parallel against the same Databricks estate:

1. Every `DataDistribution` document Atlan produces has a matching new-crawler document with identical `jrn` and identical values for attributes 9–16 (§3).
2. Every `DataAttribute` document likewise matches on `jrn` and attributes 9–18 (§4).
3. Soft-delete behavior matches: removed UC objects are marked `source.sourceStatus = DELETED`, not dropped.
4. Incremental delta runs produce the same change set Atlan's `processDeltaFromAtlan()` would.
5. A quantitative threshold (e.g. ≥99% document match on a defined Databricks scope) is agreed with stakeholders before cutover.
