# Proposal: In-House Replacement of Vendor (Atlan) Metadata Crawlers & Miners

**Prepared for:** Data Cataloging Program
**Status:** Draft for review
**Companion artifacts:** `data_compass_model.md` (34-entity catalog data model), `mvp_parity_matrix.md` (Databricks MVP acceptance checklist)

---

## 1. Executive Summary

The company catalogs its data products and metadata in an internal **Metadata Catalog DB** (MongoDB), surfaced through an internal UI. Today, technical metadata is harvested by **Atlan**, a SaaS vendor: Atlan's crawlers and miners run on-prem (a VSI running scheduled jobs), extract metadata from platforms such as Databricks, Snowflake, and Tableau, send it to Atlan's SaaS environment for processing and lineage rendering, and a separate in-house sync program reads it back into the Catalog DB.

This proposal covers **eliminating Atlan entirely** — both its crawlers/miners and its UI — and replacing the metadata-harvesting capability with in-house components that write directly to the Catalog DB. The UI is already being replaced separately by the internal UI team.

The effort is organized into **three workstreams**: Crawlers (technical metadata harvesting), Miners (lineage derivation), and Authoring/Curation (Domains, Products, glossary — owned by the internal UI team). A **build-vs-adopt evaluation** of open-source platforms (OpenMetadata, DataHub) precedes implementation as a formal decision gate. **Databricks is the pilot platform**, with **MVP parity** — faithfully reproducing the two Databricks-sourced collections the Atlan sync populates today — as the first concrete milestone.

---

## 2. Background & Current State

### 2.1 What Exists Today

- **Catalog DB** — internal MongoDB metadata catalog, with a well-defined 34-entity data model ("Data Compass"). Fed by Atlan sync, internal Systems of Record, event-monitoring systems, and Glossary systems.
- **Atlan SaaS** — vendor platform providing crawlers, miners, lineage processing, and a UI.
- **On-prem crawler/miner host** — a VSI running Atlan's vendor-provided crawler/miner application on scheduled jobs, extracting metadata from Databricks, Snowflake, Tableau, and others.
- **Atlan→Catalog sync program** — in-house service reading processed metadata back from Atlan into the Catalog DB. Four collections are populated: `DataDistribution`, `DataAttribute` (crawled from platforms) and `DataDomain`, `DataProduct` (authored in Atlan's UI), plus audit-history twins.
- **Internal UI** — already under development to replace Atlan's UI; reads from the Catalog DB.

### 2.2 Why Replace Atlan

- Eliminate ongoing SaaS licensing cost and vendor dependency.
- Remove the round-trip of company metadata out to a SaaS environment and back.
- Gain full control over crawling cadence, coverage, normalization, and the catalog data model.
- Consolidate on a single in-house catalog stack (internal UI + in-house harvesting + Catalog DB).

### 2.3 Scope of This Proposal

**In scope:** replacing Atlan's crawlers and miners with in-house components writing to the Catalog DB; the build-vs-adopt evaluation; phased platform rollout starting with Databricks.

**Out of scope (owned elsewhere):** the internal UI (UI team); the authoring surface for Domains/Products (UI team — see Workstream 3); existing System-of-Record and Glossary feeds into the Catalog DB.

---

## 3. Goals & Non-Goals

### 3.1 Goals

1. Replace Atlan's crawlers with in-house technical-metadata harvesting, Databricks first.
2. Replace Atlan's miners with in-house lineage derivation.
3. Fully serve the Catalog DB data model for data cataloging — physical/logical assets, Domains, Products, and lineage — not only the minimum needed for parity.
4. Establish MVP parity on Databricks as the first verifiable milestone and de-risking step.
5. Decommission Atlan once all platforms reach parity and all workstreams are delivered.

### 3.2 Non-Goals

- Rebuilding the catalog UI (separate, already in progress).
- Changing the Catalog DB data model itself (the 34-entity model is the fixed target contract).
- Migrating System-of-Record or Glossary feeds.

---

## 4. Workstream Structure

The effort spans three workstreams. They are sequenced and staffed independently because they require different skills and have different owners.

### Workstream 1 — Crawlers (Technical Metadata Harvesting)

Harvest technical metadata from data platforms and write the physical/logical asset entities to the Catalog DB.

- **Catalog targets:** `DataDistribution`, `DataAttribute`, and the broader physical/logical family — `Distribution`, `Dataset`, `DataModel`, `DataModelEntity`, `DataModelAttribute`, `DataService`.
- **Platforms:** Databricks (pilot) → Snowflake → Tableau → others.
- **Owner:** this project team.

### Workstream 2 — Miners (Lineage Derivation)

Derive table- and column-level lineage from platform query/access history.

- **Catalog targets:** `DataFlow`, `DataProcess`, and the seven Event entities (`DataConsumptionEvent`, `DataProductionEvent`, `DataProcessStartEvent`, `DataProcessCompleteEvent`, `DataQualityMeasurementEvent`, `DataContractBreachEvent`, `CompensatingActionEvent`).
- **Source:** Databricks Unity Catalog `system.access.*` tables / query history; Snowflake `ACCESS_HISTORY` / `QUERY_HISTORY`; Tableau Metadata API.
- **Owner:** this project team. Sequenced after the crawler pilot proves the design.
- **Note:** the hardest workstream — requires SQL parsing (CTEs, subqueries, dialect handling). Evaluate `sqlglot` / `sqllineage` rather than writing a parser.

### Workstream 3 — Authoring / Curation (Domains, Products, Business Context)

`DataDomain` and `DataProduct` are **not crawled from any platform** — they are authored constructs (created today in Atlan's UI). Likewise the semantic/glossary layer. Post-Atlan, these need an authoring surface.

- **Catalog targets:** `DataDomain`, `DataProduct`, and the business/semantic family.
- **Owner:** **internal UI team.** This workstream is owned and delivered by the UI team; this proposal records it as a coordinated dependency, not a deliverable of the crawler/miner project.
- **Dependency on this project:** the `source.*` provenance rename (Section 7.2) affects the UI team's read contract — coordination required.

---

## 5. Build vs. Adopt — Decision Gate

Before crawler implementation begins, a **time-boxed evaluation (1–2 weeks)** assesses whether to adopt an open-source metadata-ingestion framework or build connectors from scratch.

### 5.1 Options

**Adopt OpenMetadata or DataHub (for the extraction layer):**
- Mature, hardened connectors already exist for Databricks, Snowflake, Tableau.
- Solves pagination, rate limiting, auth rotation, schema drift, incremental crawl out of the box.
- The team customizes connectors and writes the normalization layer to the Catalog DB model.
- Trade-off: an open-source dependency to manage; connector output must be mapped to the 34-entity model.

**Build connectors fully in-house (greenfield):**
- Full control; output shaped exactly to the Catalog DB model; no external dependency.
- Trade-off: the team owns every platform's API changes indefinitely; re-solves problems the OSS projects already handle.

### 5.2 Evaluation Method

Run a spike against **one real Databricks workspace** with both OpenMetadata and DataHub: measure connector coverage vs. the parity matrix, effort to map output to the Catalog DB model, incremental-crawl behavior, and operational footprint. Compare against a greenfield estimate.

### 5.3 Recommendation Going In

Adopt a framework for the **extraction layer**, build **normalization and lineage resolution** in-house. The team's differentiator is the mapping to *this* catalog model and *these* lineage requirements — not re-implementing platform connectors. If organizational policy forbids OSS dependencies, that becomes the explicit, documented reason for greenfield. **The gate decision is made on evidence from the spike, not assumption.**

---

## 6. The Catalog DB Data Model (Target Contract)

The Catalog DB data model is fully documented in the companion artifact `data_compass_model.md`. Summary:

- **34 entities** across six families: Physical & logical assets (8), Business/semantic (8), Lineage & movement (4), Event records (7), Governance & classification (5), Foundational (2).
- A shared **"Managed Item" base** header (`guid`, `version`, `lifecycleStatus`, created/updated/certified audit fields, `jrn`, `owners[]`) carried by nearly every entity — the normalization anchor.
- **`jrn`** (JPMorgan Resource Name) is the universal cross-entity join key.
- The **`Node`** entity is foundational — referenced by every provider/consumer/broker relationship.

This model is the fixed target contract. Every crawler and miner output normalizes into it. It is not modified by this effort.

---

## 7. MVP Parity — First Milestone (Databricks)

MVP parity is the first verifiable milestone: the new Databricks crawler faithfully reproduces what the Atlan sync produces today for the two crawled collections. Full detail is in the companion artifact `mvp_parity_matrix.md`.

### 7.1 MVP Scope

Replace `DataDistributionSyncFromAtlanService.processDeltaFromAtlan()` and `DataAttributeSyncFromAtlanService.processDeltaFromAtlan()` for Databricks:

- **`DataDistribution`** (16 attributes) ← Databricks Unity Catalog tables/views.
- **`DataAttribute`** (18 attributes) ← Databricks Unity Catalog columns.

`DataDomain` and `DataProduct` are excluded from MVP — they belong to Workstream 3.

### 7.2 Resolved Design Decisions

| Topic | Decision |
|---|---|
| **Provenance block** | Rename `atlan.*` → neutral `source.*`. New field `source.sourceSystem = DATABRICKS`. |
| **`source.*` cutover** | **Dual-write**: the crawler writes both `atlan.*` and `source.*` during a transition window, then `atlan.*` is dropped once all readers (incl. internal UI) migrate. Decouples crawler and UI timelines. |
| **`jrn`** | The crawler calls the **existing JRN minting service** — it does not construct keys itself. |
| **`environment`** | Resolved from a **workspace-to-environment lookup table** → `PROD` / `UAT` / `DEV`. |
| **`connectionName`** | The **Databricks workspace name** (direct value). |
| **`dataType`** | **Normalized** from raw Unity Catalog type to the model's data-type enum; unrecognized → `OTHER`. |
| **`displayName`** | Equal to `name` (confirmed — no separate curated value). |
| **Unity Catalog coverage** | **All in-scope Databricks workspaces are on Unity Catalog** — no Hive metastore code path required. |
| **History collections** | History writes are triggered by the **sync service itself**; the new crawler assumes the same responsibility for `*History` documents. |
| **Soft-delete** | No hard delete. Removed UC objects are marked `source.sourceStatus = DELETED`. |

### 7.3 Remaining Open Item

- **JRN minting service — parent/child relationship & rename behavior.** How a `DataAttribute` `jrn` references its parent `DataDistribution` `jrn`, and whether a renamed source object mints a new `jrn` or retains the existing one. **Left open pending discussion with the JRN service owner.** Idempotency of minting must also be confirmed. Does not block design start but must close before the crawler write layer is finalized.

### 7.4 MVP Acceptance Definition

Running the new crawler in parallel with Atlan against the same Databricks estate:

1. Every `DataDistribution` document matches on `jrn` and all crawled attributes.
2. Every `DataAttribute` document matches on `jrn` and all crawled attributes.
3. Soft-delete behavior matches (`source.sourceStatus = DELETED`, not dropped).
4. Incremental delta runs produce the same change set as Atlan's `processDeltaFromAtlan()`.
5. A quantitative match threshold (e.g. ≥99% on a defined scope) is agreed with stakeholders before cutover.

---

## 8. Architecture

The harvesting pipeline is layered so connectors stay decoupled from the Catalog DB:

1. **Extraction layer** — per-platform connectors pull raw metadata and raw lineage signals. Connectors stage raw payloads (enables re-processing without re-crawling). Likely built on an adopted OSS framework pending the Section 5 gate.
2. **Normalization layer** — maps raw platform output to the 34-entity Catalog DB model; calls the JRN minting service; applies the `dataType` enum mapping and the workspace-to-environment lookup; emits the shared Managed Item base uniformly.
3. **Lineage resolution layer** — parses query/access history into `DataFlow` / `DataProcess` / Event entities (Workstream 2).
4. **Load layer** — writes normalized documents to the Catalog DB (MongoDB), handles dual-write of `source.*` + `atlan.*`, soft-delete, and `*History` emission.
5. **Orchestration** — a scheduler (e.g. Airflow) replaces the VSI cron jobs. Every job idempotent and independently re-runnable; one platform's failure isolated from others.
6. **Observability & reconciliation** — crawl success/failure metrics, per-run asset counts, freshness dashboards, and a recurring job diffing crawler output against expected catalog state.

---

## 9. Phased Delivery Plan

### Phase 0 — Build-vs-Adopt Gate (1–2 weeks)
OpenMetadata/DataHub spike vs. greenfield estimate. **Exit:** documented framework decision.

### Phase 1 — Discovery & Current-State Inventory
Catalog exactly what Atlan extracts today, per platform: object/property coverage, source mechanisms, lineage sources, scheduling, auth/network paths, volumes. **Exit:** inventory that serves as acceptance criteria. Snowflake/Tableau discovery can run in parallel.

### Phase 2 — Design
Connector contract, normalization contract, JRN minting integration, incremental-crawl strategy, lineage resolution design, orchestration, observability. **Exit:** ratified design contracts.

### Phase 3 — Databricks Crawler + MVP Parity
Build extraction → normalization → load for Databricks `DataDistribution` and `DataAttribute`. Run in parallel with Atlan; diff against the parity matrix. **Exit:** MVP acceptance threshold met.

### Phase 4 — Databricks Full Coverage + Miner
Extend Databricks crawler to the full physical/logical asset family; build the Databricks lineage miner (`DataFlow`, `DataProcess`, Events). **Exit:** Databricks fully served, lineage validated.

### Phase 5 — Snowflake & Tableau
Apply the proven, ratified design to Snowflake then Tableau (Snowflake may begin once Phase 2 contracts are frozen, with separate staffing). **Exit:** parity on all platforms.

### Phase 6 — Cutover & Atlan Decommission
Platform-by-platform cutover as each reaches parity. Drop `atlan.*` once all readers are on `source.*`. Decommission Atlan crawlers, miners, and SaaS subscription once all platforms pass parity **and** Workstream 3 is delivered by the UI team. **Exit:** Atlan fully removed.

---

## 10. Workstream Dependencies & Coordination

| Dependency | Between | Coordination |
|---|---|---|
| `source.*` provenance rename | Crawler project ↔ Internal UI team | Dual-write window decouples timelines; UI migrates readers `atlan.*` → `source.*` before `atlan.*` is dropped |
| JRN minting service | Crawler project ↔ JRN service owner | Resolve parent/child + rename + idempotency (Section 7.3) before crawler write layer finalized |
| Workspace-to-environment lookup | Crawler project (owns) | Define storage location and ownership for ongoing workspace onboarding; unmapped workspace must fail loudly |
| Domains/Products authoring | Crawler project ↔ Internal UI team (owns WS3) | Atlan decommission gated on WS3 delivery; both depend on the shared Catalog DB model |
| Atlan decommission | All workstreams | Final decommission only after all platforms reach parity AND WS3 delivered |

---

## 11. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Connector maintenance burden (platform APIs change) | Adopt OSS framework for extraction (Phase 0); assign explicit ongoing connector ownership |
| Lineage accuracy (column-level) | Treat the miner as the critical path; use a proven SQL parser; validate against Atlan output |
| `source.*` rename regresses UI/queries | Dual-write transition window; coordinated reader migration with UI team |
| JRN minting non-idempotency → duplicate assets | Confirm idempotency before build (Section 7.3) |
| Stale workspace-to-environment lookup mislabels assets | Clear ownership; fail-loud on unmapped workspace |
| Big-bang cutover risk | Parallel running + platform-by-platform cutover; quantitative parity threshold |
| Scope conflation (crawling vs. authoring) | Three explicit workstreams with separate owners |
| Hidden Atlan features in use beyond crawling | Phase 1 inventory captures full current-state behavior |

---

## 12. Recommended Immediate Actions

1. **Approve the three-workstream structure** and confirm the internal UI team's ownership of Workstream 3.
2. **Initiate Phase 0** — the OpenMetadata/DataHub vs. greenfield spike.
3. **Start the JRN minting service conversation** (Section 7.3) — the one external dependency with an open design question.
4. **Begin Phase 1 discovery** for Databricks in parallel with Phase 0.
5. **Engage the UI team** on the `source.*` dual-write contract and the Workstream 3 interface.

---

## Appendix — Companion Artifacts

- **`data_compass_model.md`** — full documentation of the 34-entity Catalog DB data model.
- **`mvp_parity_matrix.md`** — attribute-level Databricks MVP acceptance checklist for `DataDistribution` and `DataAttribute`.
