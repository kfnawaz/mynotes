# 20. Jira Epics Overview — Data Compass AI Operations Control Plane

## Initiative

**Data Compass AI Operations Control Plane**

## Initiative description

Build a production-grade operational control plane for Data Compass metadata and AI workflows. The platform will allow authorized users to configure, validate, version, schedule, trigger, monitor, troubleshoot, retry, cancel, reconcile, evaluate, and audit workflows and pipelines across crawlers, miners, canonical validation, AI chunk generation, vector indexing, Neo4j graph sync, reconciliation, and search/GraphRAG evaluation.

## Epic list

| Epic ID | Epic Name | Outcome |
|---|---|---|
| DC-ACP-EPIC-01 | Program Architecture and Product Definition | Approved product scope, architecture, UX, data model, APIs, and sequencing. |
| DC-ACP-EPIC-02 | Control Plane Data Model and Persistence | MongoDB collections for templates, definitions, versions, runs, pipelines, audit, reconciliation, and evaluation. |
| DC-ACP-EPIC-03 | Workflow Management Service | Backend APIs for workflow templates, definitions, versions, run creation, retry, cancel, and status. |
| DC-ACP-EPIC-04 | Template, Configuration, Validation, and Versioning | JSON schema-driven templates, config validation, version history, compare, rollback, and approval hooks. |
| DC-ACP-EPIC-05 | Execution Adapter and Orchestrator Integration | Argo/Kubernetes adapter, submission, status sync, cancel, retry, logs reference. |
| DC-ACP-EPIC-06 | Workflow Administration UI | Dashboard, workflow catalog/detail, create/edit wizard, run now, run history/detail, logs, versions, audit. |
| DC-ACP-EPIC-07 | Observability, Logs, Metrics, and Failure Taxonomy | Structured logs, metrics, trace correlation, logs viewer, failure codes, alerts. |
| DC-ACP-EPIC-08 | Security, RBAC, Audit, and Production Governance | Roles, permissions, audit events, secrets references, production controls, approvals. |
| DC-ACP-EPIC-09 | Scheduling and Runtime Operations | Schedule management, enable/disable, overlap control, missed schedules, retry/cancel operations. |
| DC-ACP-EPIC-10 | Pipeline Orchestration | Pipeline templates, definitions, DAGs, pipeline runs, stage retry, pipeline dashboard. |
| DC-ACP-EPIC-11 | AI Pipeline Operations | AI chunk, vector indexing, Neo4j graph sync, embedding migration, freshness dashboards. |
| DC-ACP-EPIC-12 | Reconciliation and Data Quality Operations | Source/MongoDB/vector/graph reconciliation, integrity checks, mismatch dashboards. |
| DC-ACP-EPIC-13 | Search and GraphRAG Evaluation Operations | Golden question sets, evaluation runs, quality metrics, regression detection. |
| DC-ACP-EPIC-14 | Connector and Workflow Template Library | Production-ready templates for Databricks, Snowflake, Tableau, Alteryx, indexers, graph builders, validation jobs. |
| DC-ACP-EPIC-15 | Production Readiness, UAT, Runbooks, and Rollout | Tests, UAT, pilot, runbooks, dashboards, alerts, handover, production readiness. |

---

# DC-ACP-EPIC-01: Program Architecture and Product Definition

## Outcome

Approved product scope, architecture, UX, data model, APIs, and sequencing.

## What is to be built

Foundational architecture and product definition for the control plane program.

## How it will be built

Document target architecture, scope, workflow vs pipeline concepts, first MVP templates, UI journeys, API/data/security/observability principles, ADRs, open questions, and backlog.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Architecture approved; MVP scope approved; first templates selected; orchestrator selected; Jira backlog ready.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-02: Control Plane Data Model and Persistence

## Outcome

MongoDB collections for templates, definitions, versions, runs, pipelines, audit, reconciliation, and evaluation.

## What is to be built

MongoDB data model for workflow templates, workflow definitions, workflow versions, runs, steps, pipeline definitions, pipeline runs, audit events, reconciliation, and evaluation results.

## How it will be built

Define MongoDB schemas/indexes, implement repositories, run status model, version model, audit persistence, and seed data.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Core collections and indexes exist; workflow versions immutable; run records reference exact versions.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-03: Workflow Management Service

## Outcome

Backend APIs for workflow templates, definitions, versions, run creation, retry, cancel, and status.

## What is to be built

Backend service that manages workflow lifecycle and run operations.

## How it will be built

Build REST APIs, permission checks, config snapshot creation, submission through adapter, and run state updates.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Create/update/view workflows; trigger manual run; retry/cancel; audit events emitted.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-04: Template, Configuration, Validation, and Versioning

## Outcome

JSON schema-driven templates, config validation, version history, compare, rollback, and approval hooks.

## What is to be built

Template-driven workflow creation and safe configuration management.

## How it will be built

Define JSON schemas, validation APIs, versioning, diff, rollback, and approval hooks.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Config schema renders; invalid configs rejected; version history/diff/rollback work.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-05: Execution Adapter and Orchestrator Integration

## Outcome

Argo/Kubernetes adapter, submission, status sync, cancel, retry, logs reference.

## What is to be built

Adapter layer to execute jobs in Argo/Kubernetes/Airflow.

## How it will be built

Define adapter interface, implement MVP adapter, normalize statuses, map run IDs, status polling/events.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Manual run submitted; status updates visible; cancel/retry/log refs available.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-06: Workflow Administration UI

## Outcome

Dashboard, workflow catalog/detail, create/edit wizard, run now, run history/detail, logs, versions, audit.

## What is to be built

User interface for workflow operations.

## How it will be built

Build React/TypeScript UI or existing Data Compass framework, schema-driven forms, grids, run detail/log viewer, RBAC-aware actions.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Users can create/edit/run workflows and inspect failures/logs.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-07: Observability, Logs, Metrics, and Failure Taxonomy

## Outcome

Structured logs, metrics, trace correlation, logs viewer, failure codes, alerts.

## What is to be built

Make failures explainable and actionable.

## How it will be built

Define logging contract, integrate logs platform, store refs/summaries, failure taxonomy, metrics dashboards.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Logs visible from run; failure stage/code shown; metrics summary and alerts configured.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-08: Security, RBAC, Audit, and Production Governance

## Outcome

Roles, permissions, audit events, secrets references, production controls, approvals.

## What is to be built

Governed controls for all actions.

## How it will be built

Define roles/permissions, implement API authorization, mask secrets, capture audit, add production reason/approval hooks.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Unauthorized actions blocked; production run requires permission/reason; secrets not exposed.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-09: Scheduling and Runtime Operations

## Outcome

Schedule management, enable/disable, overlap control, missed schedules, retry/cancel operations.

## What is to be built

Reliable scheduled operations.

## How it will be built

Build schedule APIs, orchestrator/internal schedule integration, overlap policies, missed schedule detection, UI schedule editor.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Cron schedule works; enable/disable works; overlap policy enforced.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-10: Pipeline Orchestration

## Outcome

Pipeline templates, definitions, DAGs, pipeline runs, stage retry, pipeline dashboard.

## What is to be built

End-to-end AI metadata dataflow orchestration.

## How it will be built

Build pipeline model, orchestration service, APIs, UI DAG rendering, workflow node execution, failure policy, stage retry.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Pipeline runs workflows in dependency order; DAG status visible; failed stage retry works.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-11: AI Pipeline Operations

## Outcome

AI chunk, vector indexing, Neo4j graph sync, embedding migration, freshness dashboards.

## What is to be built

Operational workflows and dashboards for AI projections.

## How it will be built

Templates for chunk generation, vector indexing, graph sync, freshness tracking, integration with AI architecture components.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Vector/graph jobs triggerable; freshness visible; failures traceable.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-12: Reconciliation and Data Quality Operations

## Outcome

Source/MongoDB/vector/graph reconciliation, integrity checks, mismatch dashboards.

## What is to be built

Prove metadata and AI projections are synchronized.

## How it will be built

Build reconciliation templates, results model, dashboards, thresholds.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Source vs MongoDB/vector/Neo4j counts visible; mismatches actionable.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-13: Search and GraphRAG Evaluation Operations

## Outcome

Golden question sets, evaluation runs, quality metrics, regression detection.

## What is to be built

Make search and GraphRAG quality measurable.

## How it will be built

Evaluation set model, runner workflow, results dashboard, integration with search/GraphRAG APIs.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Golden questions run; Top-K/citation metrics calculated; failed questions visible.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-14: Connector and Workflow Template Library

## Outcome

Production-ready templates for Databricks, Snowflake, Tableau, Alteryx, indexers, graph builders, validation jobs.

## What is to be built

Reusable template library.

## How it will be built

Define connector config schemas, images, entrypoints, outputs, validations; seed/test templates.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

Initial Databricks/Snowflake/lineage templates ready; AI/graph/reconciliation templates added later.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.

---

# DC-ACP-EPIC-15: Production Readiness, UAT, Runbooks, and Rollout

## Outcome

Tests, UAT, pilot, runbooks, dashboards, alerts, handover, production readiness.

## What is to be built

Production readiness package.

## How it will be built

UAT scenarios, readiness checklist, runbooks, dashboards/alerts, pilot, training/handover.

## What is to be achieved

A production-ready capability that contributes to the AI Operations Control Plane and reduces dependency on ad hoc scripts, direct orchestrator access, and manual support.

## Success criteria

UAT complete; pilot workflows live; runbooks approved; support ownership assigned.

## Primary dependencies

- Product scope and target architecture.
- Control-plane MongoDB model.
- RBAC/audit baseline.
- UI/API integration conventions.
