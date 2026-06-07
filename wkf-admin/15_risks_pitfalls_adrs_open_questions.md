# 15. Risks, Pitfalls, ADRs, and Open Questions

## Purpose

This page captures key risks, mitigations, architecture decisions, and open questions for the control plane program.

## Risk register

| Risk ID | Risk | Impact | Mitigation |
|---|---|---|---|
| R-001 | UI couples directly to Kubernetes/Airflow | High maintenance burden | Use Workflow Management API and orchestration adapter. |
| R-002 | Workflows and pipelines are modeled as one concept | Confusing and unscalable model | Separate workflow definitions from pipeline definitions. |
| R-003 | Config changes are not versioned | Failed runs cannot be reproduced | Create immutable workflow and pipeline versions. |
| R-004 | Run does not store config snapshot | Debugging impossible | Store version and runtime overrides on every run. |
| R-005 | Secrets stored in config | Security violation | Store only connection references. |
| R-006 | Logs stored fully in MongoDB | Cost and performance issue | Store raw logs in log platform; store refs/summaries in MongoDB. |
| R-007 | New connector requires custom UI | Poor scalability | Use schema-driven config forms. |
| R-008 | No reconciliation | AI and graph freshness cannot be trusted | Add reconciliation workflows and dashboards. |
| R-009 | No golden-question evaluation | Search regressions go unnoticed | Add evaluation workflows after indexing/graph updates. |
| R-010 | Production controls deferred too long | Audit and operational risk | Implement baseline RBAC/audit in MVP. |
| R-011 | Orchestrator-specific statuses leak into UI | Bad UX and vendor lock-in | Normalize statuses in Run State Service. |
| R-012 | Pipeline retries rerun too much or too little | Operational risk | Define retry modes and failure policies. |
| R-013 | Jobs are not idempotent | Retry can corrupt outputs | Define idempotency rules per job. |
| R-014 | Pipeline dashboard hides individual run evidence | Poor troubleshooting | Link every pipeline node to workflow run detail. |
| R-015 | Scope expands into end-user AI chat UI | Product confusion | Keep control plane as admin/operations UI. |

## Architecture Decision Records

| ADR | Decision | Rationale |
|---|---|---|
| ADR-001 | Build a control plane, not a thin orchestrator UI | Enables validation, versioning, audit, RBAC, and portability. |
| ADR-002 | Separate workflows from pipelines | Supports reuse, debuggability, and end-to-end orchestration. |
| ADR-003 | Store workflow/pipeline metadata in MongoDB | Aligns with Data Compass and flexible operational metadata. |
| ADR-004 | Use adapter layer for execution engines | Avoids lock-in and keeps APIs/UI stable. |
| ADR-005 | Use template-driven workflow creation | Scales connector onboarding without custom UI per connector. |
| ADR-006 | Store raw logs outside MongoDB | Better performance, retention, and operational fit. |
| ADR-007 | Make AI operations first-class workflows | Production AI metadata requires operational visibility and controls. |
| ADR-008 | Enforce RBAC and audit in backend services | Prevents permission bypass and supports auditability. |

## Open questions

| ID | Question | Impact | Needed By |
|---|---|---|---|
| OQ-001 | Which execution engine is approved for MVP: Argo or Kubernetes Jobs? | Blocks adapter implementation | Phase 1 |
| OQ-002 | Is Airflow required by enterprise standard? | Affects adapter roadmap | Phase 2 |
| OQ-003 | What authentication and identity provider should be used? | Blocks RBAC | Phase 1 |
| OQ-004 | What enterprise secrets platform should connection refs use? | Blocks connection design | Phase 1 |
| OQ-005 | What log platform should logs viewer integrate with? | Blocks log viewer | Phase 1 |
| OQ-006 | Which workflows are first MVP templates? | Blocks template implementation | Phase 0 |
| OQ-007 | Are production config approvals required in MVP or Phase 2? | Affects workflow UX | Phase 1 |
| OQ-008 | What are retention requirements for run history and audit events? | Affects data model | Phase 1 |
| OQ-009 | What SLAs define freshness for crawler, vector, and graph layers? | Affects alerts/dashboard | Phase 2 |
| OQ-010 | What golden question set exists for search evaluation? | Affects evaluation workflows | Phase 4 |
| OQ-011 | Should users be able to edit workflow templates from UI? | Affects admin scope | Phase 3 |
| OQ-012 | What is the max allowed manual production run scope? | Affects run controls | Phase 2 |
