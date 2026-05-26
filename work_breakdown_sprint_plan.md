# Work-Breakdown & Sprint Plan — Greenfield Crawler Build

**Project:** In-House Replacement of Atlan Crawlers/Miners
**Scope of this plan:** Workstream 1 (Crawlers) and Workstream 2 (Miners) — the greenfield build, Databricks pilot first. Workstream 3 (Domains/Products authoring) is owned by the internal UI team and not planned here.
**Approach:** Greenfield, with the extraction layer kept swappable (greenfield ↔ DataHub/OpenMetadata adapter per platform).
**Companion artifacts:** `atlan_replacement_proposal.md`, `connector_normalization_interface_contracts.md`, `databricks_uc_extraction_query_plan.md`, `harvesting_pipeline_architecture.md`, `mvp_parity_matrix.md`, `databricks_currentstate_inventory_template.md`, `data_compass_model.md`.

> **Estimates** are in relative effort (S / M / L / XL) and indicative sprint counts assuming 2-week sprints. Convert to your team's velocity. Sequencing and dependencies are the firm part; durations are indicative.

---

## 1. Guiding Build Principles

1. **Stable core before platform code.** Build the parts that never change regardless of build-vs-adopt — `Raw*` models, `Normalizer`, `CatalogLoader`, `DeltaEngine`, pipeline — and lock their contracts before any Databricks-specific code depends on them.
2. **Contract-first.** Every component is built against the interface contracts. Interfaces are implemented and tested before the concrete logic behind them.
3. **Swappability discipline.** The Databricks connector's raw-mapping sub-role populates **neutral `Raw*` models** — never a mirror of Databricks SQL output. Enforced as a code-review checklist item.
4. **Parallel-run is the proof.** Nothing is "done" until it is validated against Atlan output via the parity matrix.
5. **Vertical slice early.** Get one table flowing end-to-end (extract → normalize → load → MongoDB) as fast as possible, then broaden — depth before breadth.

---

## 2. Epic Structure

The build is eight epics. E1–E6 deliver the Databricks pilot through MVP parity; E7 extends Databricks to full coverage and the miner; E8 generalizes to Snowflake and Tableau.

| Epic | Title | Workstream | Depends on |
|---|---|---|---|
| E0 | Project Setup & Foundations | Enabling | — |
| E1 | Core Domain Models & Contracts | WS1 | E0 |
| E2 | Load Layer (MongoDB) | WS1 | E1 |
| E3 | Databricks Connector (Greenfield Extraction) | WS1 | E1 |
| E4 | Normalization Layer | WS1 | E1, external inputs |
| E5 | Delta Engine & Pipeline Orchestration | WS1 | E2, E3, E4 |
| E6 | Databricks MVP Parity & Cutover | WS1 | E5 |
| E7 | Databricks Full Coverage + Lineage Miner | WS1 + WS2 | E6 |
| E8 | Snowflake & Tableau Connectors | WS1 + WS2 | E6 (design), E7 (miner pattern) |

---

## 3. Epic Detail & Task Breakdown

### E0 — Project Setup & Foundations  *(Effort: M · ~1 sprint)*

Enabling work; no business logic.

| Task | Effort | Notes |
|---|---|---|
| E0.1 Repository, Python project scaffold (`uv`/`poetry`), lint/format/type-check config | S | Fully independent repo — not the sync service's |
| E0.2 CI pipeline — build, test, type-check, container image build & scan | M | Company CI standard |
| E0.3 Base container image; local dev container | S | One parameterized image (decision: Open Item) |
| E0.4 Kubernetes/ECS deployment skeleton — namespace, service accounts, RBAC | M | Confirm K8s vs ECS first |
| E0.5 Kubernetes Secrets convention — secret naming, mount pattern, per-instance scheme | S | Follows existing `dbx-idx-auth` convention |
| E0.6 Test framework + fixtures scaffold (`pytest`) | S | |
| E0.7 Structured logging baseline (`structlog`), `run_id` correlation | S | |

**Exit:** an empty but deployable, tested, observable skeleton.

---

### E1 — Core Domain Models & Contracts  *(Effort: M · ~1 sprint)*

The stable core. Locked before platform code depends on it.

| Task | Effort | Notes |
|---|---|---|
| E1.1 Implement shared types — enums, `SourceProvenance`, `ExtractionResult` | S | Interface contracts §2 |
| E1.2 Implement `Raw*` models — `RawDistribution`, `RawAttribute`, `RawLineageEvent` + summaries | M | **Neutral domain shapes** — the swappability contract |
| E1.3 Implement `Catalog*` models — `CatalogDistribution`, `CatalogAttribute`, `CatalogLineageEvent` | M | Interface contracts §5.3 |
| E1.4 Define `MetadataConnector`, `Normalizer`, `CatalogLoader`, `DeltaEngine`, `CrawlPipeline` Protocols | S | Protocols only — no implementations yet |
| E1.5 Define injected-dependency Protocols — `JrnMintingService`, `EnvironmentLookup`, `DataTypeMapper` | S | |
| E1.6 Interface version tag `1.0.0`; contract-change governance note | S | |
| E1.7 Contract unit tests — model construction, `ExtractionResult` invariants | M | |

**Exit:** all contracts and models implemented, versioned, unit-tested. No platform code, no I/O. This is the frozen foundation.

---

### E2 — Load Layer (MongoDB)  *(Effort: L · ~1.5 sprints)*

`CatalogLoader` implementation. Built early because it's platform-agnostic and needed for the first vertical slice.

| Task | Effort | Notes |
|---|---|---|
| E2.1 MongoDB connection management, dedicated crawler DB credential | S | |
| E2.2 `upsert_distribution` / `upsert_attribute` — upsert keyed on `jrn` | M | |
| E2.3 Dual-write — write `source.*` AND `atlan.*` blocks; toggle to disable `atlan.*` | M | **Depends on dual-write contract confirmation** |
| E2.4 Soft-delete — `source.sourceStatus = DELETED`; cascade Distribution → child Attributes | M | Never hard-delete |
| E2.5 History emission — `*History` documents on change | M | Confirm history collections exist (inventory §E) |
| E2.6 Bulk-write batching for throughput | S | |
| E2.7 Idempotency — re-running a load produces no duplicates / spurious history | M | |
| E2.8 Load-layer tests against a test MongoDB | L | Upsert, dual-write, soft-delete, history, idempotency |

**Exit:** the load layer writes valid catalog documents to MongoDB and passes all behavioral tests.
**Risk flag:** E2.3 and E2.5 depend on external confirmations (dual-write contract; history collections). Buildable behind a confirmed-config flag if answers lag.

---

### E3 — Databricks Connector (Greenfield Extraction)  *(Effort: XL · ~2 sprints)*

The first `MetadataConnector` implementation. Acquisition + raw-mapping sub-roles.

| Task | Effort | Notes |
|---|---|---|
| E3.1 IDAnywhere OAuth — token acquisition from ADFS endpoint; reuse existing client registrations | M | Architecture §3.2; secret `dbx-idx-auth` |
| E3.2 Databricks SQL connectivity — `databricks-sql-connector` against SQL warehouse | M | |
| E3.3 Acquisition: distribution inventory query (lightweight FQN + timestamp) | M | Extraction query plan §2.1 / §5.1 |
| E3.4 Acquisition: distribution detail query | M | |
| E3.5 Acquisition: attribute inventory + detail queries | M | Extraction query plan §3 |
| E3.6 Acquisition: lineage event query — `system.access.*` | L | Confirm system tables enabled |
| E3.7 Raw-mapping: SQL rows → `RawDistribution` / `RawAttribute` / `RawLineageEvent` | M | **Neutral mapping — swappability rule** |
| E3.8 `CrawlScope` enforcement — include/exclude catalogs/schemas; auto-exclude system schemas | S | Mirror Atlan's exclude-by-hierarchy/regex |
| E3.9 Connector exception handling — typed `ConnectorError` hierarchy | S | |
| E3.10 Connector tests — against a real Databricks workspace + recorded fixtures | L | Per interface contracts §9 test surface |

**Exit:** the Databricks connector extracts metadata and lineage and emits valid `Raw*` models for a real workspace.
**Risk flag:** E3.6 depends on `system.access.*` being enabled (inventory §D).

---

### E4 — Normalization Layer  *(Effort: L · ~1.5 sprints)*

`Normalizer` implementation. Project core IP — stays in-house regardless of build-vs-adopt.

| Task | Effort | Notes |
|---|---|---|
| E4.1 `JrnMintingService` client — integrate the existing minting service | M | **Depends on JRN service interface confirmation** |
| E4.2 `EnvironmentLookup` — workspace-to-environment lookup; fail-loud on unmapped | S | Confirm lookup location/ownership |
| E4.3 `DataTypeMapper` — UC raw type → model `DataTypeEnum`; `OTHER` fallback | M | Extraction query plan §3.3; complete type table from inventory |
| E4.4 `normalize_distribution` — `RawDistribution` → `CatalogDistribution` | M | Assembles `source.*` provenance block |
| E4.5 `normalize_attribute` — `RawAttribute` → `CatalogAttribute`; parent `jrn` reference | M | |
| E4.6 `normalize_lineage_event` — `RawLineageEvent` → `CatalogLineageEvent` | M | |
| E4.7 Complex-type handling decision — `struct`/`map`/`variant` (collapse vs flatten) | S | Phase 2 design call |
| E4.8 Normalization tests — determinism (same input → same output), enum mapping, provenance | L | |

**Exit:** the normalizer deterministically converts `Raw*` to `Catalog*` documents.
**Risk flag:** E4.1 depends on the JRN minting service interface + idempotency confirmation — the project's key open external dependency.

---

### E5 — Delta Engine & Pipeline Orchestration  *(Effort: L · ~1.5 sprints)*

Wires extraction → normalization → load; adds delta and scheduling.

| Task | Effort | Notes |
|---|---|---|
| E5.1 `DeltaEngine` — distribution & attribute delta (new/updated/deleted) | L | Extraction query plan §5; hybrid full-FQN + watermark |
| E5.2 Raw payload staging — write/read S3, partitioned by platform/instance/run | M | |
| E5.3 Watermark store — per-instance watermark persistence | M | Decision: Airflow DB vs MongoDB collection |
| E5.4 `CrawlPipeline` — orchestrate the full sequence per instance | L | Interface contracts §8 |
| E5.5 Instance registry — config source enumerating crawlable instances | M | Architecture §3.7 |
| E5.6 Airflow DAGs — one DAG per instance; containerized task operators | L | |
| E5.7 Failure isolation — per-instance independence; watermark only on success | M | |
| E5.8 Pipeline integration tests — full run, idempotency, failure isolation | L | |

**Exit:** a scheduled, idempotent, instance-isolated pipeline runs the full Databricks crawl end-to-end.

---

### E6 — Databricks MVP Parity & Cutover  *(Effort: L · ~1.5 sprints)*

Validation against Atlan and the first production cutover.

| Task | Effort | Notes |
|---|---|---|
| E6.1 Parallel-run harness — new crawler alongside Atlan, same workspace | M | |
| E6.2 Parity diff tool — compare `DataDistribution`/`DataAttribute` vs Atlan output per parity matrix | L | mvp_parity_matrix.md §8 |
| E6.3 Reconciliation job — recurring crawler-output vs expected-state diff | M | Observability layer |
| E6.4 Parity gap remediation — close diffs found in E6.2 | M-L | Size unknown until E6.2 runs |
| E6.5 Observability dashboards — per-run counts, durations, failures (Grafana) | M | |
| E6.6 Alerting — crawl failure, coverage drop, reconciliation mismatch | S | |
| E6.7 Cutover — meet the agreed parity threshold; switch Databricks off Atlan | M | Stakeholder sign-off |

**Exit:** the new crawler is the source of truth for Databricks `DataDistribution`/`DataAttribute`; Atlan's Databricks crawler is retired. **MVP achieved.**

---

### E7 — Databricks Full Coverage + Lineage Miner  *(Effort: XL · ~2.5 sprints)*

Extends beyond MVP to the full physical/logical family and lineage (Workstream 2).

| Task | Effort | Notes |
|---|---|---|
| E7.1 Extend connector/normalizer — `Dataset`, `DataModel`, `DataModelEntity`, `DataService` | L | Data model §5.1 |
| E7.2 Lineage resolution layer — `RawLineageEvent` → `DataFlow` / `DataProcess` | XL | Hardest component |
| E7.3 SQL parsing — `sqlglot` (bench vs `sqllineage`); CTE/subquery/dialect handling | L | |
| E7.4 FQN → `jrn` resolution — lineage edges resolve to crawler-populated catalog | M | |
| E7.5 Event entity emission — the seven Event entities | L | Data model §5.4 |
| E7.6 Lineage parity validation vs Atlan lineage output | L | |
| E7.7 Column-level lineage accuracy baseline | M | |

**Exit:** Databricks is fully served — all in-scope asset entities + lineage — and Atlan's Databricks lineage miner is retired.

---

### E8 — Snowflake & Tableau Connectors  *(Effort: XL · ~3 sprints, parallelizable)*

Generalizes the proven design to the remaining platforms. New connectors only — core, normalization, load, delta, pipeline are reused unchanged.

| Task | Effort | Notes |
|---|---|---|
| E8.1 Snowflake connector — Custom OAuth; `ACCOUNT_USAGE`/`INFORMATION_SCHEMA`; per-instance identity | XL | Architecture §3.3 |
| E8.2 Snowflake `DataTypeMapper` + raw-mapping | M | |
| E8.3 Snowflake lineage — `ACCESS_HISTORY`/`QUERY_HISTORY` | L | |
| E8.4 Tableau connector — JWT Bearer; Metadata API (GraphQL); per-instance identity | XL | Architecture §3.4 |
| E8.5 Tableau raw-mapping + lineage | L | |
| E8.6 Per-platform parity validation & cutover | L | |
| E8.7 Full Atlan decommission — once all platforms parity-passed AND WS3 delivered | M | Coordinated with UI team |

**Exit:** all platforms served in-house; Atlan fully decommissioned.

---

## 4. Sequencing & Critical Path

```
E0 ──► E1 ──┬──► E2 ───────────────┐
            │                      │
            ├──► E3 ───────────────┼──► E5 ──► E6 ──► E7 ──► E8
            │                      │
            └──► E4 ───────────────┘
                 ▲
                 │ (external: JRN service, env lookup)
```

- **E1 is the gate.** Nothing concrete proceeds until the core contracts are locked.
- **E2, E3, E4 run in parallel** after E1 — different developers, no inter-dependency. This is the main parallelization opportunity.
- **E5 is the join point** — needs E2, E3, E4 complete.
- **Critical path:** E0 → E1 → (longest of E2/E3/E4, which is E3) → E5 → E6. E3 (Databricks connector, XL) is the critical-path item — staff it strongest.
- **E7 and E8** follow E6; E8 Snowflake and Tableau tracks can run in parallel with separate staffing once the E7 miner pattern exists.

**Indicative timeline to MVP (E6 exit):** ~9–10 sprints if E2/E3/E4 parallelize well; longer if single-threaded. E7 adds ~2.5; E8 adds ~3.

---

## 5. External Dependencies — Must Resolve Before the Dependent Task

| Dependency | Blocks | Needed by (latest) |
|---|---|---|
| JRN minting service — interface, idempotency, parent/child, rename, auth | E4.1, E4.5 | Start of E4 |
| Dual-write contract — `atlan.*` readers, field mapping, indexes, exit criteria | E2.3 | Mid-E2 |
| History collections — existence & trigger semantics | E2.5 | Mid-E2 |
| Workspace-to-environment lookup — location & ownership | E4.2 | Start of E4 |
| `system.access.*` enabled on the metastore | E3.6, E7.2 | Start of E3 lineage work |
| K8s vs ECS decision | E0.4 | Start of E0 |
| Phase 1 inventory (current-state) | E6.2 acceptance criteria | Before E6 |

> The JRN minting service is the highest-risk dependency — it sits on the critical path via E4. Start that conversation immediately (it is not the project team's to answer).

---

## 6. Definition of Done — Per Layer

A component is done when:
- It implements its interface contract exactly.
- It has unit tests covering the behavioral invariants in the contract.
- For connectors: it passes the minimum test surface in interface contracts §9.
- For the pipeline: it is idempotent and re-runnable, proven by test.
- It emits structured logs and metrics.
- MVP-scope work additionally: it passes the parity diff against Atlan output.

---

## 7. Swappability Checkpoints

To keep the extraction layer genuinely swappable, these are explicit review gates — not optional:

1. **E1.2 review** — `Raw*` models are neutral domain shapes; no Databricks/DataHub-specific field names. Sign-off required.
2. **E3.7 review** — the Databricks raw-mapping populates `Raw*` without leaking SQL-result structure; implementation-specific extras go to `properties`.
3. **E8 review** — adding Snowflake/Tableau required no change to `Raw*`, `Normalizer`, `CatalogLoader`, `DeltaEngine`, or pipeline. If it did, the contract leaked — investigate.

A passing E8 with zero downstream changes is the proof the architecture's swappability claim held.

---

## 8. Immediate Next Actions

1. **Start E0** — repo, CI, deployment skeleton. No blockers.
2. **Open the JRN minting service conversation** — critical-path dependency for E4 (questions already itemized).
3. **Open the dual-write contract conversation with the UI team** — needed for E2.3.
4. **Confirm K8s vs ECS** — needed for E0.4.
5. **Begin Phase 1 inventory** — runs in parallel; needed for E6 acceptance criteria.
