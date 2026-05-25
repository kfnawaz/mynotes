# Phase 1 — Databricks Current-State Inventory Template

**Project:** In-House Replacement of Atlan Crawlers/Miners
**Purpose:** Capture exactly what Atlan extracts from Databricks today. The completed inventory becomes the **acceptance criteria** for the new Databricks crawler and miner — the new components are correct when they reproduce everything recorded here.
**Companion artifacts:** `atlan_replacement_proposal.md`, `mvp_parity_matrix.md`, `data_compass_model.md`.

> **How to use this template:** Fill every section from *evidence* — Atlan connection configuration, Atlan crawl logs/run history, the on-prem VSI job definitions, the existing sync service code, and direct inspection of the Catalog DB. Where a value is assumed rather than verified, mark it `[ASSUMED]`. Inventory runs in parallel with the Phase 0 build-vs-adopt spike.

---

## Section A — Databricks Estate

Establishes the full crawl surface. Confirmed: all in-scope workspaces are on Unity Catalog.

### A.1 Workspaces

| Workspace name | Workspace URL/ID | AWS account | Region | Environment (`PROD`/`UAT`/`DEV`) | In Atlan crawl scope? | Notes |
|---|---|---|---|---|---|---|
| | | | | | | |

### A.2 Unity Catalog Metastores

| Metastore | Region | Workspaces attached | Notes |
|---|---|---|---|
| | | | |

### A.3 Catalog / Schema Scope

| Workspace | Catalog | Schemas crawled | Schemas excluded | Exclusion reason |
|---|---|---|---|---|
| | | | | |

### A.4 Estate Volumes (sizing for crawl design)

| Metric | Count | Source of figure |
|---|---|---|
| Total catalogs in scope | | |
| Total schemas in scope | | |
| Total tables in scope | | |
| Total views in scope | | |
| Total columns in scope | | |
| Largest single schema (object count) | | |

### A.5 Open Items — Section A

- Confirm every workspace maps to an entry in the workspace-to-environment lookup table.
- Confirm no workspace is on the legacy Hive metastore (proposal assumes UC-only).
- Identify any workspace onboarded but not yet crawled by Atlan.

---

## Section B — Atlan Connection & Crawler Configuration

What Atlan is configured to do — extracted from Atlan's connection setup and crawler settings.

### B.1 Atlan Connection(s) for Databricks

| Connection name | Workspaces covered | Auth method | Service principal / account | Permission scope | Notes |
|---|---|---|---|---|---|
| | | | | | |

> `connectionName` in the Catalog DB = the Databricks workspace name (per parity matrix). Record how Atlan's connection maps to workspaces so the new crawler stamps the same value.

### B.2 Crawler Configuration

| Setting | Atlan value | Notes |
|---|---|---|
| Crawl mode (full / incremental) | | |
| Object types crawled | | tables, views, columns, schemas, catalogs, … |
| Object types excluded | | e.g. functions, volumes, ML models |
| Include/exclude filters (catalog/schema) | | |
| Metadata depth (descriptions, tags, properties) | | |
| Sampling/profiling enabled? | | data profiling vs. metadata-only |

### B.3 Crawl Schedule & Execution

| Item | Value | Source |
|---|---|---|
| Schedule (cron / frequency) | | VSI job definition |
| Trigger mechanism | | scheduled / manual / event |
| Typical crawl duration | | Atlan run history |
| Crawl host | | the on-prem VSI |
| Concurrency / parallelism | | |
| Failure/retry behavior | | |

### B.4 Open Items — Section B

- Obtain read-only export of the Atlan connection config for the record.
- Confirm whether any crawl filters silently exclude objects that *should* be cataloged.
- Confirm the service principal's permission scope — the new crawler needs an equivalent.

---

## Section C — Crawled Metadata Coverage (Object & Attribute Level)

The core of the inventory. For every object type, record every attribute Atlan extracts, its Databricks Unity Catalog source, and its Catalog DB target. This is the direct acceptance checklist for Workstream 1.

### C.1 Table / View → `DataDistribution`

For each attribute: confirmed crawled-by-Atlan, its UC source, its Catalog DB target field. Cross-check against `mvp_parity_matrix.md` §3.

| Catalog DB attribute | Crawled by Atlan? | Databricks UC source | Transform/normalization | Verified against live data? | Notes |
|---|---|---|---|---|---|
| `source.sourceGuid` | | | | | |
| `source.sourceQualifiedName` | | `catalog.schema.object` | | | |
| `source.sourceStatus` | | presence at source | ACTIVE / DELETED | | |
| `source.sourceTypeName` | | `information_schema.tables.table_type` | | | |
| `source.syncStatus` | | crawler-owned | | | |
| `source.syncTime` | | crawler-owned | | | |
| `source.sourceSystem` | | constant | `DATABRICKS` | | new field |
| `name` | | `information_schema.tables.table_name` | | | |
| `displayName` | | = `name` | none | | confirmed in proposal |
| `description` | | `information_schema.tables.comment` | | | |
| `jrn` | | JRN minting service | | | |
| `environment` | | workspace → lookup | PROD/UAT/DEV | | |
| `connectionName` | | workspace name | | | |
| `databaseName` | | UC schema name | | | |
| `catalogName` | | UC catalog name | | | |
| *(any attribute Atlan crawls not listed above)* | | | | | **flag — model gap or out-of-scope** |

### C.2 Column → `DataAttribute`

Cross-check against `mvp_parity_matrix.md` §4.

| Catalog DB attribute | Crawled by Atlan? | Databricks UC source | Transform/normalization | Verified against live data? | Notes |
|---|---|---|---|---|---|
| `source.sourceGuid` | | | | | |
| `source.sourceQualifiedName` | | `catalog.schema.table.column` | | | |
| `source.sourceStatus` | | presence at source | | | |
| `source.sourceTypeName` | | constant `COLUMN` | | | |
| `source.syncStatus` | | crawler-owned | | | |
| `source.syncTime` | | crawler-owned | | | |
| `source.sourceSystem` | | constant | `DATABRICKS` | | |
| `name` | | `information_schema.columns.column_name` | | | |
| `displayName` | | = `name` | none | | |
| `description` | | `information_schema.columns.comment` | | | |
| `dataType` | | `information_schema.columns.data_type` | normalize → model enum | | unrecognized → `OTHER` |
| `jrn` | | JRN minting service | references parent | | |
| `environment` | | workspace → lookup | | | |
| `connectionName` | | workspace name | | | |
| `databaseName` | | UC schema name | | | |
| `tableName` | | `information_schema.columns.table_name` | | | |
| `catalogName` | | UC catalog name | | | |
| *(any attribute Atlan crawls not listed above)* | | | | | **flag** |

### C.3 Extended Asset Coverage (Workstream 1, beyond MVP)

The proposal extends Workstream 1 past MVP to the full physical/logical family. Record whether Atlan supplies metadata for these and from where.

| Catalog model entity | Does Atlan populate it today? | Databricks source | Currently via which sync collection? | Notes |
|---|---|---|---|---|
| `Distribution` | | | | MVP via `DataDistribution` |
| `DataModelAttribute` | | | | MVP via `DataAttribute` |
| `Dataset` | | | | |
| `DataModel` | | | | |
| `DataModelEntity` | | | | |
| `DataService` | | | | warehouse/connection metadata |

### C.4 Open Items — Section C

- For every attribute marked "any attribute Atlan crawls not listed above" — decide: model gap, or out-of-scope. Record the decision.
- Verify each UC source mapping against a live workspace, not assumption.
- Confirm the raw UC `data_type` values present in the estate so the normalization mapping table is complete.

---

## Section D — Lineage / Miner Coverage (Workstream 2)

What lineage Atlan's miners derive today, and from which Databricks sources. Acceptance criteria for Workstream 2.

### D.1 Lineage Sources

| Lineage source | Used by Atlan today? | Granularity | Notes |
|---|---|---|---|
| UC `system.access.table_lineage` | | table-level | |
| UC `system.access.column_lineage` | | column-level | |
| Query history | | | |
| Notebook / job lineage | | | |
| Manual / declared lineage | | | created in Atlan UI |

### D.2 Lineage Coverage & Catalog Targets

| Catalog model entity | Populated by Atlan miner today? | Databricks source | Notes |
|---|---|---|---|
| `DataFlow` | | | observed lineage |
| `DataProcess` | | | transformations |
| `DataConsumptionEvent` | | | |
| `DataProductionEvent` | | | |
| `DataProcessStartEvent` | | | |
| `DataProcessCompleteEvent` | | | |
| `DataQualityMeasurementEvent` | | | |
| `DataContractBreachEvent` | | | |
| `CompensatingActionEvent` | | | |

### D.3 Lineage Characteristics

| Item | Value | Notes |
|---|---|---|
| Lineage granularity achieved (table / column) | | |
| Lineage refresh frequency | | |
| Known lineage gaps / unsupported patterns | | e.g. dynamic SQL, external tools |
| Cross-platform lineage (Databricks ↔ other) | | |

### D.4 Open Items — Section D

- Confirm whether any lineage in the catalog is *manually declared* in Atlan (would be lost on decommission — needs a Workstream 3 home).
- Establish a column-level lineage accuracy baseline from current Atlan output.

---

## Section E — Catalog DB Sync Behavior

How crawled metadata currently lands in the Catalog DB. Defines load-layer and delta behavior the new crawler must replicate.

### E.1 Sync Services

| Collection | Sync service / method | Trigger | Notes |
|---|---|---|---|
| `DataDistribution` | `DataDistributionSyncFromAtlanService.processDeltaFromAtlan()` | | |
| `DataAttribute` | `DataAttributeSyncFromAtlanService.processDeltaFromAtlan()` | | |
| `DataDomain` | `DataDomainSyncFromAtlanService.syncDataFromAtlan()` | | Workstream 3 — not crawler |
| `DataProduct` | `DataProductSyncFromAtlanService.syncDataFromAtlan()` | | Workstream 3 — not crawler |

### E.2 Delta & Lifecycle Behavior

| Behavior | Current implementation | New crawler must replicate? | Notes |
|---|---|---|---|
| Incremental delta detection | | yes | how are changes identified |
| Soft-delete (`source.sourceStatus = DELETED`) | | yes | no hard delete |
| Upsert vs. insert | | yes | |
| History write trigger | sync service itself | yes | `*History` documents |
| Conflict handling vs. SoR / event feeds | | confirm | catalog has multiple feeders |

### E.3 History Collections

| History collection | Confirmed to exist? | Triggered by | What it records | Notes |
|---|---|---|---|---|
| `DataDistributionHistory` | | sync service | parent attrs + historical timestamp | |
| `DataAttributeHistory` | | sync service | parent attrs + historical timestamp | |

### E.4 Open Items — Section E

- Confirm existence and exact trigger semantics of `DataDistributionHistory` / `DataAttributeHistory`.
- Confirm how the dual-write transition (`atlan.*` + `source.*`) interacts with existing indexes.
- Document any MongoDB indexes on `atlan.*` paths that the UI team must migrate.

---

## Section F — Operational & Access Footprint

What the new crawler needs to run, drawn from the current Atlan setup.

### F.1 Access & Credentials

| Item | Current (Atlan) | New crawler requirement | Notes |
|---|---|---|---|
| Databricks auth method | | | |
| Service principal / account | | | new one likely required |
| Unity Catalog permissions needed | | | `USE CATALOG`, `USE SCHEMA`, `SELECT` on system tables, etc. |
| SQL warehouse used for crawl | | | owned warehouses already available |
| Catalog DB (MongoDB) write credentials | | | |
| JRN minting service access | | | |

### F.2 Network & Infrastructure

| Item | Current | New crawler | Notes |
|---|---|---|---|
| Crawl host | on-prem VSI | new orchestration host | |
| Network path host → Databricks | | | |
| Network path host → Catalog DB | | | |
| Scheduler / orchestration | VSI cron | e.g. Airflow | |

### F.3 Open Items — Section F

- Provision a dedicated Databricks service principal for the new crawler (do not reuse Atlan's).
- Confirm UC system-table read permissions for the miner (`system.access.*`).
- Confirm network paths from the new orchestration host.

---

## Section G — Hidden / Out-of-Band Atlan Behavior

Catch capabilities in use beyond straightforward crawling — these are common cutover surprises.

| Capability | In use? | Detail | Disposition (replace / drop / Workstream 3) |
|---|---|---|---|
| Custom metadata / tags applied in Atlan | | | |
| Glossary terms linked to Databricks assets | | | |
| Manual lineage declared in Atlan UI | | | |
| Classifications / data-sensitivity tags | | | |
| Atlan-side data quality rules | | | |
| Saved searches / reports downstream consumers depend on | | | |
| API integrations pulling from Atlan directly | | | |

> Anything `In use? = yes` that is not replaced by a workstream is a **cutover risk** — escalate to the proposal's risk register.

---

## Section H — Inventory Sign-Off

| Item | Owner | Status | Date |
|---|---|---|---|
| Section A — Databricks Estate | | | |
| Section B — Atlan Connection & Config | | | |
| Section C — Crawled Metadata Coverage | | | |
| Section D — Lineage / Miner Coverage | | | |
| Section E — Catalog DB Sync Behavior | | | |
| Section F — Operational Footprint | | | |
| Section G — Hidden Atlan Behavior | | | |

**Inventory accepted as crawler/miner acceptance criteria by:** ______________  **Date:** __________

---

## Appendix — Evidence Sources Checklist

Complete the inventory from these, not from memory:

- [ ] Atlan connection configuration (Databricks)
- [ ] Atlan crawler settings / scope filters
- [ ] Atlan crawl run history / logs
- [ ] On-prem VSI job definitions (schedule, triggers)
- [ ] Existing sync service source code (the four `*SyncFromAtlanService` classes)
- [ ] Direct Catalog DB inspection (sample documents, indexes, history collections)
- [ ] Databricks Unity Catalog `information_schema` (live verification)
- [ ] Workspace-to-environment lookup table
- [ ] Databricks platform inventory (workspaces, accounts, warehouses)
