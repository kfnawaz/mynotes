# 21. Detailed Jira Stories — Data Compass AI Operations Control Plane

## Story template

```markdown
## Objective

## Background

## Scope

### In Scope

### Out of Scope

## What needs to be built

## How it should be built

## Dependencies

## Acceptance Criteria

## Test / Evidence

## Related Confluence
```


---

# Epic 01 — Program Architecture and Product Definition

## DC-ACP-001 — Define Control Plane Product Scope

### Objective

Define the product boundary for the Data Compass AI Operations Control Plane.

### What needs to be built

A scope document that separates workflow administration, pipeline operations, AI operations, reconciliation, evaluation, and audit from the business-facing Data Compass search/chat UI.

### How it should be built

Create Confluence pages describing included/excluded capabilities, personas, MVP scope, and phased roadmap.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Scope page is approved; MVP and later phases are clearly separated.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-002 — Document Production-Grade Target Architecture

### Objective

Create the target architecture for the control plane.

### What needs to be built

Architecture document with UI, APIs, services, stores, execution engines, jobs, observability, and security components.

### How it should be built

Use text and Mermaid diagrams to show component interactions and runtime flows.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Architecture diagram and component descriptions are approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-003 — Select MVP Orchestration Engine

### Objective

Decide whether MVP execution uses Argo Workflows or Kubernetes Jobs.

### What needs to be built

An ADR comparing options and selecting the MVP engine.

### How it should be built

Evaluate enterprise availability, support model, DAG needs, scheduling, retries, logs, and operational maturity.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- ADR is approved and adapter target is identified.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-004 — Identify First MVP Workflow Templates

### Objective

Select initial workflow templates for MVP.

### What needs to be built

A list of two to four workflow templates to implement first.

### How it should be built

Recommend Databricks crawler, Snowflake crawler, one lineage miner, and canonical validation.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Template list is approved and owners are assigned.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 02 — Control Plane Data Model and Persistence

## DC-ACP-020 — Create MongoDB Collection Schemas

### Objective

Define and implement control-plane MongoDB collections.

### What needs to be built

Collections for templates, definitions, versions, runs, steps, pipelines, audit, reconciliation, and evaluation.

### How it should be built

Create schema definitions, indexes, and repository layer.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Collections and indexes are created; sample seed data can be loaded.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-021 — Implement Workflow Template Persistence

### Objective

Store reusable workflow templates.

### What needs to be built

Persistence for template ID, type, platform, image, entrypoint, config schema, defaults, overrides, and outputs.

### How it should be built

Build template repository and seed loader.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Templates can be created, read, and filtered by type/platform/status.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-022 — Implement Workflow Definition and Version Persistence

### Objective

Store configured workflow instances and immutable versions.

### What needs to be built

Workflow definition and workflow version collections with versioning behavior.

### How it should be built

Every workflow config update creates a new immutable version.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Workflow v1 is created; update creates v2; old versions remain unchanged.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-023 — Implement Run and Step Persistence

### Objective

Store workflow run and step status.

### What needs to be built

Run and step collections with status, timing, output counts, failure fields, and log references.

### How it should be built

Implement repositories and state update methods.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Run transitions work; step records attach; output/failure data is stored.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-024 — Implement Pipeline Definition and Run Persistence

### Objective

Store pipeline DAG definitions and runs.

### What needs to be built

Collections for pipeline definitions, versions, runs, and run nodes.

### How it should be built

Persist DAG nodes, dependencies, failure policy, and node status.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Pipeline definition can be saved; pipeline run tracks node statuses.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 03 — Workflow Management Service

## DC-ACP-030 — Build Workflow Catalog API

### Objective

Return searchable and filterable workflow list.

### What needs to be built

GET /workflows with filters, pagination, sorting, last run summary, and available actions.

### How it should be built

Query MongoDB workflow definitions and join last run summary.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Filters, pagination, permissions, and last-run summaries work.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-031 — Build Workflow Detail API

### Objective

Return full workflow detail.

### What needs to be built

GET /workflows/{workflowId} with metadata, current version, schedule, config summary, recent runs, and permissions.

### How it should be built

Aggregate workflow definition, current version, schedule, last runs, and user permissions.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Workflow detail renders current version, schedule, owner, and recent run summary.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-032 — Build Create Workflow API

### Objective

Allow authorized users to create workflows from templates.

### What needs to be built

POST /workflows that validates template, config, permissions, and connection refs before creating definition and v1.

### How it should be built

Use Template/Config service for validation and Audit service for event capture.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Workflow is created; invalid config is rejected; audit event is emitted.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-033 — Build Update Workflow API

### Objective

Allow config updates through new versions.

### What needs to be built

PUT /workflows/{workflowId} that creates new version and supports production approval hooks.

### How it should be built

Require change reason and enforce RBAC; preserve old versions.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- New version is created; audit event emitted; PROD controls are honored.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-034 — Build Manual Run API

### Objective

Trigger workflow run from the UI.

### What needs to be built

POST /workflows/{workflowId}/runs with version, run mode, runtime overrides, and reason.

### How it should be built

Validate permissions/version/overrides; create run; submit to adapter; audit.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Run is created/submitted; overrides stored; PROD reason required.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-035 — Build Retry and Cancel APIs

### Objective

Support operational recovery.

### What needs to be built

POST /runs/{runId}/retry and POST /runs/{runId}/cancel.

### How it should be built

Check retryability, status, permissions, and call adapter.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Retry/cancel calls adapter and emits audit.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 04 — Template, Configuration, Validation, and Versioning

## DC-ACP-040 — Define JSON Schema for Databricks Crawler Template

### Objective

Create config schema for Databricks metadata crawler.

### What needs to be built

Schema fields for workspace, catalogs, schemas, tables, include/exclude options, target MongoDB, resource profile, and log level.

### How it should be built

Define JSON schema and validation warnings for broad source scope.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Schema validates and UI can render the form.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-041 — Define JSON Schema for Snowflake Crawler Template

### Objective

Create config schema for Snowflake metadata crawler.

### What needs to be built

Schema fields for account, database, schema/table filters, role/warehouse ref, and outputs.

### How it should be built

Define JSON schema and expected output entity mapping.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Schema validates and expected outputs are documented.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-042 — Build Config Validation API

### Objective

Validate workflow configuration before save or run.

### What needs to be built

POST /config-validation/workflow returning structured errors and warnings.

### How it should be built

Implement schema validation and custom validation hooks.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Invalid config is rejected with structured messages.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-043 — Build Version Compare API

### Objective

Show differences between workflow versions.

### What needs to be built

API returns changed fields, old/new values, and masks sensitive values.

### How it should be built

Compare stored version snapshots.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- UI can render version diff safely.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-044 — Build Rollback Version API

### Objective

Allow rollback by creating new version from old version.

### What needs to be built

API that copies selected prior version into a new current version with reason and audit.

### How it should be built

Never delete or mutate old versions.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Rollback creates new version and emits audit.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 05 — Execution Adapter and Orchestrator Integration

## DC-ACP-050 — Define Orchestration Adapter Interface

### Objective

Create engine-neutral interface for execution.

### What needs to be built

Interface for submit, status, steps, logs, cancel, retry, and error mapping.

### How it should be built

Document normalized statuses and engine-specific mapping.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Interface is approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-051 — Implement MVP Orchestrator Adapter

### Objective

Execute workflow runs using selected MVP orchestrator.

### What needs to be built

Argo or Kubernetes adapter with labels, status lookup, and log refs.

### How it should be built

Use selected SDK/API and inject workflowId/runId labels.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Manual run executes and returns orchestratorRunId.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-052 — Implement Status Synchronization

### Objective

Keep control-plane status in sync with execution engine.

### What needs to be built

Polling or event-based updates to run/step records.

### How it should be built

Normalize raw orchestrator status into control-plane status.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Terminal statuses are captured; stuck-run detection is planned or implemented.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-053 — Pass Config Snapshot to Job Runtime

### Objective

Ensure job receives exact config for run.

### What needs to be built

Pass config snapshot securely, plus runId/workflowId and runtime overrides.

### How it should be built

Use environment variables, mounted config, or secure runtime payload; secrets by reference only.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Job logs show runId and uses stored config snapshot.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 06 — Workflow Administration UI

## DC-ACP-060 — Build UI Shell and Navigation

### Objective

Create UI shell and routes.

### What needs to be built

Dashboard, Workflows, Runs, Failures, Logs, Audit routes.

### How it should be built

Use existing Data Compass UI framework or React/TypeScript shell.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Routes exist and identity/permissions are available.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-061 — Build Dashboard Screen

### Objective

Show operational health.

### What needs to be built

Widgets for running workflows, recent failures, success/failure counts, freshness, and links.

### How it should be built

Use aggregated BFF endpoint.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Dashboard links to run and workflow detail.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-062 — Build Workflow Catalog Screen

### Objective

List and filter workflows.

### What needs to be built

Table, filters, search, status badges, last run status, RBAC-aware actions.

### How it should be built

Consume GET /workflows.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Workflow list is usable and filters work.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-063 — Build Workflow Detail Screen

### Objective

Show workflow details and tabs.

### What needs to be built

Overview, Configuration, Runs, Versions, Audit tabs.

### How it should be built

Consume workflow detail, versions, runs, audit APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Current version and last run are visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-064 — Build Create/Edit Workflow Wizard

### Objective

Create and edit workflows from templates.

### What needs to be built

Template selection, details, schema-driven form, validation, review, save.

### How it should be built

Use JSON schema form renderer and validation API.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- User can create workflow from template.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-065 — Build Run Now Modal

### Objective

Trigger safe manual runs.

### What needs to be built

Run mode, overrides, PROD reason, validation, submit.

### How it should be built

Use Manual Run API and show audit warning for PROD.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Run starts and redirects to run detail.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-066 — Build Run History and Run Detail Screens

### Objective

Monitor and troubleshoot runs.

### What needs to be built

Run list, run detail tabs, steps, logs, metrics, errors, config snapshot, audit.

### How it should be built

Consume runs, logs, metrics APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Failed run can be investigated and retried if allowed.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 07 — Observability, Logs, Metrics, and Failure Taxonomy

## DC-ACP-070 — Define Structured Logging Contract

### Objective

Standardize logs emitted by jobs.

### What needs to be built

Required log fields, correlation IDs, step IDs, sensitive data rules.

### How it should be built

Publish logging contract and update crawler/miner runtime wrapper.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Jobs can emit runId/workflowId/stepId logs.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-071 — Integrate Logs Viewer with Log Platform

### Objective

Show logs for a run.

### What needs to be built

Logs query by runId, filters by level/step, link to raw platform, permission checks.

### How it should be built

Build Observability Service adapter over enterprise log platform.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Logs are visible in UI.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-072 — Implement Failure Classification

### Objective

Classify failures into standard error codes.

### What needs to be built

Map failures to category, stage, code, retryable flag, recommended action.

### How it should be built

Implement mapping in job wrapper or Observability Service.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- UI shows failure classification.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-073 — Build Metrics Summary View

### Objective

Show run metrics and output counts.

### What needs to be built

Records read/written, assets changed, duration, errors, step metrics.

### How it should be built

Persist summary in run records and expose via API.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Metrics visible in run detail.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 08 — Security, RBAC, Audit, and Production Governance

## DC-ACP-080 — Define Roles and Permissions

### Objective

Create RBAC model.

### What needs to be built

Roles, permissions, environment controls, log access rules.

### How it should be built

Document model with security stakeholders.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- RBAC model approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-081 — Implement Backend Authorization Checks

### Objective

Enforce permissions in APIs.

### What needs to be built

Checks for view/create/edit/run/retry/cancel/log/audit actions.

### How it should be built

Add policy/RBAC service integration and tests.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Unauthorized calls rejected.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-082 — Implement Audit Event Capture

### Objective

Audit key actions.

### What needs to be built

Create/edit/run/retry/cancel/enable/disable events with before/after and reason.

### How it should be built

Integrate Audit service in mutating APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Audit events are searchable.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-083 — Implement Secret Reference Handling

### Objective

Prevent secret exposure.

### What needs to be built

Connection refs only, masked sensitive fields, redaction guidance.

### How it should be built

Use approved secrets manager references.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- No raw secrets stored or shown.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 09 — Scheduling and Runtime Operations

## DC-ACP-090 — Build Schedule Model and APIs

### Objective

Support scheduled workflow runs.

### What needs to be built

Store cron/timezone/enabled state; expose next run.

### How it should be built

Implement schedules repository and APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Schedule can be saved/displayed.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-091 — Implement Schedule Execution

### Objective

Trigger scheduled runs.

### What needs to be built

Integrate internal scheduler or orchestrator-native schedules.

### How it should be built

Scheduled run records created with triggerType SCHEDULED.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Scheduled runs appear in run history.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-092 — Implement Overlap Control

### Objective

Prevent unsafe concurrent runs.

### What needs to be built

Support BLOCK_OVERLAP and QUEUE_NEXT behavior.

### How it should be built

Add workflow locks or status checks before scheduling/manual run.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Overlapping production runs blocked or queued.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 10 — Pipeline Orchestration

## DC-ACP-100 — Define Pipeline Data Model

### Objective

Model pipelines and DAGs.

### What needs to be built

Pipeline templates, definitions, versions, runs, nodes, failure policy.

### How it should be built

Define schemas and persistence.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Pipeline schemas approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-101 — Build Pipeline Orchestration Service

### Objective

Execute workflow DAGs.

### What needs to be built

Start root nodes, unlock dependencies, stop/continue per policy, track status.

### How it should be built

Use Workflow Management Service to start child workflow runs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Pipeline executes workflows in dependency order.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-102 — Build Pipeline Catalog and Detail UI

### Objective

Show pipelines and DAG view.

### What needs to be built

Pipeline list, detail, DAG rendering, run details.

### How it should be built

Consume pipeline APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Pipeline DAG status visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-103 — Implement Failed Stage Retry

### Objective

Retry failed pipeline stage.

### What needs to be built

Retry failed node only and emit audit; optionally retry downstream.

### How it should be built

Implement retry policy in Pipeline Orchestration Service.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Failed stage retry works.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 11 — AI Pipeline Operations

## DC-ACP-110 — Create AI Chunk Generation Workflow Template

### Objective

Operationalize AI chunk generation.

### What needs to be built

Template supports entity type scope, full/incremental modes, output counts.

### How it should be built

Create template, schema, expected outputs, and run summary fields.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Chunk generation run visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-111 — Create Vector Indexing Workflow Template

### Objective

Operationalize vector embedding/upsert.

### What needs to be built

Template supports full/incremental/by JRN/by entity type; capture model version and failures.

### How it should be built

Create vector indexer template and output metrics.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Vector indexing run visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-112 — Create Neo4j Graph Sync Workflow Template

### Objective

Operationalize graph projection.

### What needs to be built

Template supports full rebuild/incremental sync; capture node/edge counts.

### How it should be built

Create graph sync template and failure taxonomy.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Graph sync run visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-113 — Build AI Pipeline Freshness Dashboard

### Objective

Show health of AI projections.

### What needs to be built

Last chunk generation, vector update, graph sync; stale status by platform/environment.

### How it should be built

Build aggregate freshness query and UI widgets.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Freshness visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 12 — Reconciliation and Data Quality Operations

## DC-ACP-120 — Define Reconciliation Checks

### Objective

Define consistency checks.

### What needs to be built

Source vs MongoDB, MongoDB vs chunks, chunks vs vector, MongoDB vs Neo4j.

### How it should be built

Document check logic, thresholds, scope fields.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Checks documented.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-121 — Build Reconciliation Job Template

### Objective

Run reconciliation as workflow.

### What needs to be built

Template supports scope, stores results, thresholds.

### How it should be built

Implement job template and result model.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Reconciliation run creates result records.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-122 — Build Reconciliation Dashboard

### Objective

Show mismatch results.

### What needs to be built

Counts, failed checks, detail links, trends.

### How it should be built

Create UI and API for reconciliation results.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Mismatches visible/actionable.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 13 — Search and GraphRAG Evaluation Operations

## DC-ACP-130 — Define Golden Question Set Model

### Objective

Store evaluation questions and expected results.

### What needs to be built

Question text, category, expected source JRNs/results, thresholds.

### How it should be built

Define collections and import format.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Question set stored.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-131 — Build Evaluation Runner Workflow

### Objective

Run golden questions against search/GraphRAG.

### What needs to be built

Execute exact, semantic, lineage tests; store actuals; calculate pass/fail.

### How it should be built

Implement workflow template and evaluation service.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Evaluation run works.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-132 — Build Evaluation Dashboard

### Objective

Show search quality results.

### What needs to be built

Top-K, citation coverage, failed questions, trends.

### How it should be built

Create dashboard and result APIs.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Quality metrics visible.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 14 — Connector and Workflow Template Library

## DC-ACP-140 — Build Databricks Crawler Template

### Objective

Production-ready Databricks crawler template.

### What needs to be built

Complete schema, image, entrypoint, outputs, pilot workflow.

### How it should be built

Seed template and validate through pilot run.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Template usable.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-141 — Build Snowflake Crawler Template

### Objective

Production-ready Snowflake crawler template.

### What needs to be built

Complete schema, connection requirements, outputs, pilot workflow.

### How it should be built

Seed template and validate through pilot run.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Template usable.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-142 — Build Lineage Miner Template

### Objective

Lineage miner template.

### What needs to be built

Lookback window, source scope, DataFlow/DataProcess outputs.

### How it should be built

Seed template and run pilot.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Template usable.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.


---

# Epic 15 — Production Readiness, UAT, Runbooks, and Rollout

## DC-ACP-150 — Create UAT Test Plan

### Objective

Define UAT scenarios.

### What needs to be built

Personas, scenarios, pass/fail criteria.

### How it should be built

Write UAT plan and review with operators.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- UAT plan approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-151 — Create Operational Runbooks

### Objective

Create support runbooks.

### What needs to be built

Workflow failure, pipeline failure, reconciliation mismatch, vector/graph failure runbooks.

### How it should be built

Create runbook pages and link to UI failure categories.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Runbooks approved.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-152 — Execute Pilot Rollout

### Objective

Pilot selected workflows.

### What needs to be built

Onboard three workflows and one pipeline; collect operator feedback.

### How it should be built

Execute pilot and track findings.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Pilot complete.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.

## DC-ACP-153 — Complete Production Readiness Review

### Objective

Obtain production readiness approval.

### What needs to be built

Validate security, observability, runbooks, support owners, and pilot issues.

### How it should be built

Run production readiness review.

### Dependencies

- Product scope and target architecture.
- Control-plane data model, where applicable.
- RBAC and audit baseline for mutating actions.
- API/UI conventions.

### Acceptance criteria

- Production readiness signed off.
- The implementation is linked to the relevant Confluence page.
- Any mutating action emits an audit event if applicable.
- Test evidence is attached to the Jira story.

### Test / Evidence

- Unit or integration test result, where applicable.
- Screenshot/API response/log sample for UI/API stories.
- Demo note or run evidence for workflow/orchestration stories.
