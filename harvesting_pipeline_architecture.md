# Harvesting Pipeline — Architecture & Tech Stack

**Project:** In-House Replacement of Atlan Crawlers/Miners
**Purpose:** Defines the architecture, technology stack, and per-platform authentication design for the metadata harvesting pipeline (crawlers + miners).
**Companion artifacts:** `atlan_replacement_proposal.md`, `connector_normalization_interface_contracts.md`, `databricks_uc_extraction_query_plan.md`, `mvp_parity_matrix.md`, `data_compass_model.md`.

## Foundational Decisions (settled)

| Decision | Choice | Rationale |
|---|---|---|
| Execution model | **Containerized on Kubernetes / ECS** | Owned AWS accounts; per-platform isolation; horizontal scale for the large Databricks estate |
| Language | **Python** | Matches the OSS ingestion frameworks under evaluation and the drafted interface contracts |
| Integration with Atlan-sync service | **Fully independent** | Different lifecycle — the crawler replaces the sync; no shared codebase or coupling |

> "Fully independent" means a separate repo, separate deployment, separate ownership. The crawler and the legacy sync service share only the **Catalog DB (MongoDB)** as a destination — never code. The crawler re-implements its own `CatalogLoader` per the interface contracts.

---

## 1. Architecture Overview

Six layers, each a distinct concern, wired by the orchestrator. The boundaries match the interface contracts already defined.

```
                          ┌─────────────────────────────────────┐
                          │          ORCHESTRATION              │
                          │   (Airflow — schedules & wires)     │
                          └──────────────────┬──────────────────┘
                                             │ launches containerized tasks
            ┌────────────────────────────────┼────────────────────────────────┐
            ▼                                ▼                                ▼
   ┌─────────────────┐            ┌─────────────────┐             ┌─────────────────┐
   │  1. EXTRACTION  │            │  1. EXTRACTION  │             │  1. EXTRACTION  │
   │   Databricks    │            │    Snowflake    │             │     Tableau     │
   │   connector     │            │    connector    │             │    connector    │
   └────────┬────────┘            └────────┬────────┘             └────────┬────────┘
            │  RawDistribution / RawAttribute / RawLineageEvent             │
            └────────────────────────────┬─────────────────────────────────┘
                                         ▼
                          ┌─────────────────────────────┐
                          │   RAW PAYLOAD STAGING       │
                          │   (S3 — raw, re-processable)│
                          └──────────────┬──────────────┘
                                         ▼
            ┌────────────────────────────┴────────────────────────────┐
            ▼                                                         ▼
   ┌─────────────────┐                                     ┌─────────────────────┐
   │ 2. NORMALIZATION│                                      │ 3. LINEAGE          │
   │   Raw → Catalog │                                      │    RESOLUTION       │
   │   model         │                                      │  RawLineageEvent →  │
   │  + JRN mint     │                                      │  DataFlow/Process   │
   │  + dtype enum   │                                      │  + SQL parsing      │
   │  + env lookup   │                                      └──────────┬──────────┘
   └────────┬────────┘                                                 │
            │  CatalogDistribution / CatalogAttribute                   │ CatalogLineageEvent
            └────────────────────────────┬──────────────────────────────┘
                                         ▼
                          ┌─────────────────────────────┐
                          │   4. LOAD                   │
                          │   Upsert → MongoDB          │
                          │   dual-write source.*/atlan.*│
                          │   soft-delete + history     │
                          └──────────────┬──────────────┘
                                         ▼
                          ┌─────────────────────────────┐
                          │   Catalog DB (MongoDB)      │
                          └─────────────────────────────┘

   ┌──────────────────────────────────────────────────────────────────────┐
   │  6. OBSERVABILITY — metrics, logs, traces, reconciliation (all layers)│
   └──────────────────────────────────────────────────────────────────────┘
```

---

## 2. Layer-by-Layer Design

### 2.1 Extraction Layer

**Responsibility:** Connect to a platform, extract raw metadata and raw lineage signals, emit `Raw*` models. No transformation, no catalog knowledge, no writes (per the connector contract).

**Design:**
- One containerized connector per platform, each implementing `MetadataConnector`.
- Connectors are **stateless across runs** — all run state (watermarks) is passed in by the orchestrator and persisted by the pipeline, not held in the connector.
- Two-tier extraction: cheap inventory queries (FQN + timestamp) feed the delta engine; full detail queries run only for changed objects.
- Each connector run is a discrete container task — a Databricks crawl and a Snowflake crawl are separate pods/tasks with no shared process.

**Tech:**
- **Databricks:** `databricks-sql-connector` (Python) against an owned SQL warehouse — executes the `information_schema` / `system.*` queries from the extraction query plan.
- **Snowflake:** `snowflake-connector-python`.
- **Tableau:** `tableau-api-lib` or direct Metadata API (GraphQL) calls via `requests`.
- If the Phase 0 spike adopts an OSS framework, the framework's `Source` is wrapped to satisfy `MetadataConnector` — the wrapper lives in the extraction layer.

### 2.2 Raw Payload Staging

**Responsibility:** Persist raw extracted payloads before normalization so normalization can be re-run without re-querying source platforms.

**Design:**
- Each connector run writes its raw output to object storage, partitioned by `platform / workspace / run_id`.
- Staged payloads are retained for a defined window (e.g. 30 days) for replay and debugging.
- Normalization reads from staging, not directly from connectors — this decouples extraction timing from normalization.

**Tech:** **Amazon S3** (owned AWS accounts). Raw payloads as JSON or Parquet. Lifecycle policy expires old partitions.

### 2.3 Normalization Layer

**Responsibility:** Map `Raw*` models to `Catalog*` models. Call the JRN minting service, apply `dataType` enum normalization, resolve `environment` via the workspace lookup, assemble the `source.*` provenance block.

**Design:**
- Implements the `Normalizer` contract — stateless per record.
- Injected dependencies (`JrnMintingService`, `EnvironmentLookup`, `DataTypeMapper`) per the interface contracts — swappable, testable.
- Platform-agnostic: the same normalization code serves all platforms because connectors already produced uniform `Raw*` models.
- Emits the shared Managed Item base uniformly.

**Tech:** Pure Python. No framework needed — this is the project's core IP and stays in-house regardless of the build-vs-adopt decision.

### 2.4 Lineage Resolution Layer

**Responsibility:** Convert `RawLineageEvent` streams into `DataFlow`, `DataProcess`, and event entities. Resolve object FQNs to catalog `jrn`s. Parse SQL where lineage must be derived from query text.

**Design:**
- Reads lineage events incrementally (event-timestamp watermark).
- Resolves source/target FQNs against the crawler-populated catalog — lineage referencing an un-crawled object is flagged.
- Aggregates repeated edges into `DataFlow` records; sets `precision` (TABLE vs COLUMN).
- SQL parsing for view definitions / query text uses a proven parser — **not** hand-written.

**Tech:**
- **`sqlglot`** for SQL parsing — strong dialect support (Databricks/Spark SQL, Snowflake), actively maintained, handles CTEs and subqueries. `sqllineage` (built on sqlglot/sqlparse) is an alternative if higher-level lineage extraction is preferred — evaluate both in Phase 2.
- Pure Python aggregation logic.

### 2.5 Load Layer

**Responsibility:** Write `Catalog*` documents to MongoDB. Owns upsert, dual-write (`source.*` + `atlan.*`), soft-delete cascade, history emission.

**Design:**
- Implements the `CatalogLoader` contract.
- All writes are upserts keyed on `jrn`.
- Dual-writes `source.*` and `atlan.*` during the transition window; configurable so `atlan.*` can be switched off later.
- Soft-delete only — never removes documents; cascades from Distribution to child Attributes.
- Emits `*History` documents on change.
- Idempotent — re-running a load produces no duplicate documents or spurious history.

**Tech:** **`pymongo`** against the Catalog DB. Bulk-write operations for throughput. Connection from a dedicated crawler MongoDB credential (not shared with the sync service).

### 2.6 Orchestration Layer

**Responsibility:** Schedule crawl runs, wire the layers per run, isolate failures, manage watermarks.

**Design:**
- One DAG per platform (or per platform+workspace for the large Databricks estate).
- Each layer is a task; tasks run as containerized operators.
- **Failure isolation:** one workspace/platform failure does not block others — independent DAG runs.
- Watermark persisted only on full-run success.
- Every task idempotent and independently re-runnable.

**Tech:** **Apache Airflow**, deployed on the same Kubernetes/ECS cluster. Tasks run via `KubernetesPodOperator` (or ECS equivalent) so each crawl step is an isolated container. Airflow replaces the legacy VSI cron jobs.

### 2.7 Observability Layer

**Responsibility:** Make every run measurable, diagnosable, and verifiable against expected state.

**Design — three pillars plus reconciliation:**
- **Metrics:** per-run object counts (new/updated/deleted) per platform, query durations, normalization failures, lineage events processed, watermark values.
- **Logs:** structured (JSON) logs from every layer, correlated by `run_id`.
- **Traces:** optional distributed tracing across the layer pipeline for latency analysis.
- **Reconciliation:** a scheduled job diffs crawler output against expected catalog state and alerts on coverage drops — this is what catches silent regressions.

**Tech:**
- Metrics: **Prometheus** + **Grafana** dashboards.
- Logs: structured logging via `structlog`, shipped to the company's existing log platform (CloudWatch / ELK — match what exists).
- Tracing: **OpenTelemetry** if distributed tracing is wanted.
- Alerting: on crawl failure, coverage drop, and reconciliation mismatch.

---

## 3. Authentication — Per-Platform Design

The authentication design below reflects the **actual mechanisms in use by the current Atlan workflows** (evidenced by the Atlan workflow configurations — see §3.6). The new crawler reproduces these same mechanisms; it does not introduce new auth patterns. All secrets follow the established **Kubernetes Secrets** convention. Secrets are never in code, images, or config.

### 3.1 Secrets Management (cross-cutting)

The current Atlan secure-agent workflows store all platform credentials as **Kubernetes Secrets** (observed secret names: `dbx-idx-auth`, `sf-auth-token-ccb`, `tb-jwt-token`). The new crawler follows the same convention — it already runs on Kubernetes/ECS, so this is the natural and consistent choice.

| Concern | Approach |
|---|---|
| Secret storage | **Kubernetes Secrets** — the established company convention for the existing secure-agent crawlers. One secret per platform instance |
| Secret naming | Follow the existing convention (`<platform>-<purpose>-<scope>`, e.g. `dbx-idx-auth`); extend per-instance where multiple instances exist |
| Secret injection | Mounted into connector containers at runtime as files or env vars; never baked into images |
| Rotation | Credentials rotated per the issuing system's policy; the connector re-reads the secret each run so rotation requires no redeploy |
| Least privilege | Each platform identity scoped to read-only metadata access only |
| Audit | All credential use logged; crawler identities distinct and traceable |

> If the company later standardizes on a dedicated secrets manager (e.g. HashiCorp Vault, AWS Secrets Manager), the connector's secret-retrieval step is a single injected component and can be swapped without touching connector logic. Kubernetes Secrets is the current-state-aligned default.

### 3.2 Databricks Authentication — IDAnywhere OAuth

The current Atlan Databricks crawler and lineage miner authenticate via **JPMC IDAnywhere** — a corporate machine-to-machine OAuth2 flow against an internal ADFS token endpoint. The Atlan UI labels the auth toggle "PAT," but the operative credential is the IDAnywhere-issued token obtained through the agent's custom configuration.

| Item | Design |
|---|---|
| Auth mechanism | **IDAnywhere OAuth2** — `manager_type: id_anywhere`. The crawler obtains a bearer token from the internal ADFS token endpoint, then uses it against Databricks |
| Client registration | **Reuse the existing IDAnywhere client registrations** — `client_id: CC-…-PROD` and `resource: JPMC:URI:RS-…-PROD`. No new registration required (confirmed) |
| Token endpoint | The internal ADFS OAuth2 token URL (`https://…jpmorganchase.com/adfs/oauth2/token`) |
| Secret | Kubernetes Secret — current secret name `dbx-idx-auth`. Holds the IDAnywhere client credentials |
| Crawler extraction | **REST API** method against the Databricks workspace (the Atlan crawler uses REST API; JDBC is the alternative) |
| Lineage extraction | **System Tables** — Databricks secure-agent lineage supports System Tables only; the miner reads `system.access.*`. Each lineage workflow carries an explicit **SQL Warehouse ID** |
| Permissions | Read-only: `USE CATALOG` / `USE SCHEMA` on in-scope objects; `SELECT` on `information_schema` / `system.information_schema.*`; `SELECT` on `system.access.table_lineage` and `system.access.column_lineage`; usage on the designated SQL warehouse |
| Scope | Read-only — no write/modify rights on any catalog object |

### 3.3 Snowflake Authentication — Custom OAuth

The current Atlan Snowflake crawler uses **Custom OAuth** (the Atlan auth options also include Keypair, OKTA SSO, Microsoft Entra ID, Basic — Custom OAuth is the one in use).

| Item | Design |
|---|---|
| Auth mechanism | **Custom OAuth** — an OAuth token presented to Snowflake. Aligns with the corporate OAuth pattern |
| Identity | A Snowflake service user with a scoped read-only **role** (current role pattern `PROD_…_FR`) |
| Warehouse | An explicit Snowflake **warehouse** is designated per instance (current pattern `PROD_…_WH`) |
| Secret | Kubernetes Secret — current secret name `sf-auth-token-ccb` |
| Permissions | Role granted `USAGE` on databases/schemas; `SELECT` on `SNOWFLAKE.ACCOUNT_USAGE` / `INFORMATION_SCHEMA` for metadata; access to `ACCESS_HISTORY` / `QUERY_HISTORY` for the miner |
| Scope | Read-only role; no DML/DDL grants |
| Note | `ACCOUNT_USAGE` has latency (minutes to hours) vs. live `INFORMATION_SCHEMA` — the connector accounts for this in delta logic |

### 3.4 Tableau Authentication — JWT Bearer

The current Atlan Tableau crawler uses **JWT Bearer** authentication (a connected-app flow; the Atlan options also include Basic and PAT).

| Item | Design |
|---|---|
| Auth mechanism | **JWT Bearer** — a connected-app JWT. Config carries Username, Client ID, Secret ID, Secret Value, and Site |
| Identity | A Tableau service account with a read-only site role |
| Site | Tableau site identifier (current value `ccb`) |
| Secret | Kubernetes Secret — current secret name `tb-jwt-token` |
| API | Tableau **Metadata API** (GraphQL) for asset/lineage metadata; REST API for site/project enumeration |
| Transport | SSL enabled; optional SSL certificate path |
| Permissions | Read-only site role; Metadata API access enabled on the Tableau instance |
| Note | Confirm the Metadata API is enabled on each Tableau Online account — not on by default on all deployments |

### 3.5 Internal Service Authentication

| Service | Auth |
|---|---|
| JRN minting service | Service-to-service auth per the company standard (mTLS / OAuth client credentials) — confirm with the JRN service owner |
| Catalog DB (MongoDB) | Dedicated crawler database user, scoped to write the crawler-owned collections only |
| Workspace-to-environment lookup | Read access to wherever the lookup lives (config store / DB collection) |

### 3.6 Evidence — Current Atlan Workflow Configurations

This authentication design is derived from the current Atlan secure-agent workflow configurations, recorded here for the project:

| Platform | Atlan workflow | Auth in use | Secret (K8s) | Extraction method | Notable config |
|---|---|---|---|---|---|
| Databricks (crawler) | `atlan-databricks-1730897974` (`odessa-gold-prod-us-east-1`) | IDAnywhere OAuth (`id_anywhere`) | `dbx-idx-auth` | REST API | Exclude-by-hierarchy + exclude-by-regex asset selection; internal schemas regex-excluded |
| Databricks (lineage) | `atlan-databricks-lineage-1731007349` | IDAnywhere OAuth | `dbx-idx-auth` | Self-Deployed Runtime — System Tables only | Explicit SQL Warehouse ID; Extraction Catalog Type (Default / Cloned) |
| Snowflake (crawler) | `atlan-snowflake-1774316611` (`ccbpawsglobalvps`) | Custom OAuth | `sf-auth-token-ccb` | Self-Deployed Runtime | Explicit Role (`PROD_…_FR`) and Warehouse (`PROD_…_WH`) |
| Tableau (crawler) | `atlan-tableau-1773247977` (`prod-useast-b`) | JWT Bearer | `tb-jwt-token` | Self-Deployed Runtime | Site `ccb`; Client ID + Secret ID + Secret Value; SSL enabled |

Common pattern across all: Self-Deployed Runtime / secure agent running **inside the company environment on Kubernetes**, with credentials in **Kubernetes Secrets**. This validates both the K8s/ECS execution decision and the Kubernetes Secrets convention — the new crawler replaces the agent, not the runtime model.

### 3.7 Multi-Instance Crawling

Each platform has **multiple instances** to crawl — multiple Databricks workspaces, multiple Snowflake instances, multiple Tableau Online accounts. The architecture treats an *instance* as the unit of configuration and scheduling, not the platform.

**Design principles:**

- **Instance registry.** A configuration registry enumerates every crawlable instance: platform, instance identifier, host, environment (`PROD`/`UAT`/`DEV`), the Kubernetes Secret name holding its credentials, and platform-specific parameters (Databricks SQL Warehouse ID; Snowflake role + warehouse; Tableau site). This registry is the single source of truth for what gets crawled. Adding an instance is a registry entry — no code change.
- **One connector configuration per instance.** `ConnectorConfig` (from the interface contracts) is populated per instance from the registry. The same connector code serves every instance of a platform; only the config differs.
- **Per-instance credentials.** Each instance has its own Kubernetes Secret, named per the established convention extended with an instance discriminator (e.g. `dbx-idx-auth-<workspace>`). No credential is shared across instances — preserves isolation and per-instance audit.
- **Per-instance scheduling and isolation.** The orchestrator runs one DAG (or DAG run) per instance. A failure crawling one Databricks workspace does not block another workspace, another Snowflake instance, or Tableau. Watermarks are tracked per instance.
- **Per-instance environment resolution.** The workspace-to-environment lookup keys on instance identifier; `environment` (`PROD`/`UAT`/`DEV`) and `connectionName` are resolved per instance, so assets from different instances are correctly labeled and never collide.
- **Identity scope.** Where one corporate IDAnywhere registration covers multiple Databricks workspaces, the same `client_id`/`resource` may be reused across those instances — but each instance still has its own secret entry and its own connector config. Confirm per-platform whether one identity spans instances or each instance needs a distinct one (Phase 1 inventory).

**Scale implication:** the orchestration layer fans out across the full instance set. Total crawl concurrency = (instances × per-instance partitioning). The instance registry plus per-instance DAGs make this horizontal scaling a configuration concern, not a code concern.

---

## 4. Tech Stack Summary

| Layer / Concern | Technology | Notes |
|---|---|---|
| Language | Python 3.11+ | Matches OSS frameworks and interface contracts |
| Execution | Containers on Kubernetes or ECS | Owned AWS accounts |
| Orchestration | Apache Airflow | `KubernetesPodOperator` / ECS operator per task |
| Databricks extraction | `databricks-sql-connector` | Against owned SQL warehouse |
| Snowflake extraction | `snowflake-connector-python` | Key-pair auth |
| Tableau extraction | `tableau-api-lib` / Metadata API | GraphQL |
| SQL parsing (lineage) | `sqlglot` (+ evaluate `sqllineage`) | Dialect-aware |
| Catalog DB writes | `pymongo` | Bulk upserts |
| Raw staging | Amazon S3 | Partitioned, lifecycle-expired |
| Secrets | Kubernetes Secrets | Established convention (`dbx-idx-auth`, etc.); per-instance |
| Metrics | Prometheus + Grafana | Per-run dashboards |
| Logging | `structlog` → CloudWatch / ELK | Structured, `run_id`-correlated |
| Tracing (optional) | OpenTelemetry | Cross-layer latency |
| Packaging | Docker | One image per connector, or one parameterized image |
| Dependency mgmt | `uv` or `poetry` | Reproducible builds |
| Testing | `pytest` | Connector test surface per interface contracts §9 |
| CI/CD | Company standard | Build, test, scan, publish images |

> If the Phase 0 spike adopts OpenMetadata or DataHub, `openmetadata-ingestion` or `acryl-datahub` joins the stack **only in the extraction layer** — wrapped behind the `MetadataConnector` interface. Every other layer is unaffected.

---

## 5. Deployment & Runtime Model

- **One container image per connector** (or a single parameterized image selecting the connector at runtime).
- **Airflow** schedules DAGs; each DAG run launches connector → normalization → load as containerized tasks.
- **Horizontal scale:** the large Databricks estate is partitioned — multiple parallel tasks by catalog or schema — sized against the volume figures from the Phase 1 inventory.
- **Failure isolation:** platform/workspace DAGs are independent; one failure is contained.
- **Watermarks** stored in a small state store (Airflow's metadata DB, or a dedicated collection) — persisted only on successful completion.
- **Environments:** the pipeline itself runs in a non-prod environment first; crawl targets (PROD/UAT/DEV Databricks workspaces) are independent of where the crawler runs.

---

## 6. How This Maps to the Interface Contracts

| Architecture layer | Interface contract |
|---|---|
| Extraction | `MetadataConnector`, `ConnectorConfig`, `CrawlScope` |
| (extraction output) | `RawDistribution`, `RawAttribute`, `RawLineageEvent` |
| Normalization | `Normalizer`, injected `JrnMintingService` / `EnvironmentLookup` / `DataTypeMapper` |
| (normalization output) | `CatalogDistribution`, `CatalogAttribute`, `CatalogLineageEvent` |
| Lineage resolution | consumes `RawLineageEvent`, produces `CatalogLineageEvent` |
| Load | `CatalogLoader`, `LoadResult` |
| Delta (within orchestration) | `DeltaEngine`, `DeltaSet` |
| Orchestration | `CrawlPipeline`, `CrawlRun`, `CrawlRunResult` |

The architecture is the runtime realization of the contracts — every layer above corresponds to a contract already specified.

---

## 7. Open Items for Phase 2 Design

1. **Kubernetes vs. ECS** — pick one based on what the company already operates; both satisfy the design.
2. **One image vs. per-connector images** — decide packaging granularity.
3. **`sqlglot` vs. `sqllineage`** — bench both on real Databricks/Snowflake query text in Phase 2.
4. **Watermark store** — Airflow metadata DB vs. a dedicated MongoDB collection. Watermarks are tracked **per instance** (§3.7).
5. **Identity scope per platform** — confirm whether one IDAnywhere registration / Snowflake identity / Tableau identity spans multiple instances or each instance needs a distinct one (Phase 1 inventory). Databricks: existing IDAnywhere registrations are reused.
6. **OSS framework decision** — Phase 0 spike result determines whether the extraction layer wraps a framework or calls platform SQL/APIs directly.
7. **Log/metrics platform** — confirm and match the company's existing observability stack rather than introducing parallel tooling.
8. **JRN minting service auth** — confirm the service-to-service mechanism with the JRN service owner.
9. **Instance registry location** — decide where the multi-instance registry lives (config repo, MongoDB collection, config service) and who owns onboarding new instances (§3.7).
10. **Tableau Metadata API** — confirm it is enabled on every Tableau Online account in scope.
