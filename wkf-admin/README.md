# Data Compass AI Operations Control Plane — Updated Program Package

This package updates the prior Workflow Administration plan into a holistic, production-grade **Data Compass AI Operations Control Plane** program.

The control plane is broader than crawler/miner administration. It is the operational command center for the full Data Compass AI metadata pipeline:

```text
Source platforms
→ crawler / miner workflows
→ MongoDB canonical metadata load
→ canonical validation
→ AI chunk generation
→ vector embedding and reindexing
→ Neo4j graph projection
→ reconciliation
→ search / GraphRAG evaluation
→ operational evidence, audit, and support
```

## Files

| File | Purpose |
|---|---|
| `00_program_index.md` | Package index and recommended reading order |
| `01_executive_summary_and_program_vision.md` | Program vision, goals, users, and principles |
| `02_production_grade_target_architecture.md` | Target architecture and interface diagrams |
| `03_control_plane_scope_and_capability_model.md` | Full capability model and product boundaries |
| `04_component_architecture_and_responsibilities.md` | Component responsibilities and ownership |
| `05_data_model_and_state_management.md` | MongoDB data model, state model, and status lifecycle |
| `06_api_and_interface_contracts.md` | Control-plane APIs and interface contracts |
| `07_ui_ux_design_and_user_journeys.md` | UI/UX screens, journeys, and wireframes |
| `08_workflow_pipeline_orchestration_design.md` | Workflow vs pipeline orchestration and execution adapters |
| `09_configuration_templates_versioning_validation.md` | Template-driven config, validation, versioning, and approval |
| `10_logging_metrics_observability.md` | Logging, metrics, traces, dashboards, alerts |
| `11_security_rbac_audit_governance.md` | RBAC, audit, production controls, secrets, approvals |
| `12_ai_pipeline_operations_reconciliation_evaluation.md` | AI operations, vector/graph/reconciliation/evaluation workflows |
| `13_implementation_phases_sequencing.md` | Sequencing, releases, gates, and first-sprint recommendation |
| `14_operating_model_runbooks_production_readiness.md` | Operating model, support tiers, runbooks, UAT, rollout |
| `15_risks_pitfalls_adrs_open_questions.md` | Risks, ADRs, and open questions |
| `epics/20_epics_overview.md` | Detailed epics |
| `stories/21_detailed_jira_stories.md` | Detailed Jira stories |

Recommended module name:

> **Data Compass AI Operations Control Plane**
