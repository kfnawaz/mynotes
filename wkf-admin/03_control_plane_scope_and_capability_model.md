# 03. Control Plane Scope and Capability Model

## Purpose

This page defines the functional scope of the Data Compass AI Operations Control Plane and organizes the program into capabilities.

## Product boundary

The control plane is an operational and administrative module. It manages workflows and pipelines that keep Data Compass metadata, AI chunks, vector indexes, graph projections, and evaluation results current and reliable.

It is not the Data Compass end-user search/chat interface.

## Capability model

```text
Data Compass AI Operations Control Plane
├── 1. Operational Dashboard
├── 2. Workflow Administration
├── 3. Workflow Template Management
├── 4. Configuration Management and Versioning
├── 5. Scheduling and Manual Triggering
├── 6. Run Monitoring and Troubleshooting
├── 7. Pipeline Operations
├── 8. AI Pipeline Operations
├── 9. Reconciliation and Data Quality
├── 10. Search and GraphRAG Evaluation
├── 11. Logging, Metrics, and Alerts
├── 12. Security, RBAC, and Audit
├── 13. Notifications and Escalation
├── 14. Production Change Governance
└── 15. Operations Runbooks and Evidence
```

## Capabilities

| Capability | Purpose | Key functions | MVP? |
|---|---|---|---:|
| Operational Dashboard | Overall health | Running jobs, failures, freshness, stale pipelines, quality summary | Yes |
| Workflow Administration | Manage individual jobs | Create, edit, run, retry, cancel, view runs/logs | Yes |
| Workflow Templates | Reusable job blueprints | Connector templates, config schemas, output expectations | Yes, seeded |
| Config + Versioning | Safe reproducibility | Validate, save versions, compare, rollback, approvals | Yes |
| Scheduling + Triggers | Timed/ad hoc execution | Cron, enable/disable, run now, manual overrides | Yes |
| Run Troubleshooting | Recover failures | Step status, logs, metrics, config snapshot, error code | Yes |
| Pipeline Operations | End-to-end flows | DAGs, pipeline runs, stage retry, dependency execution | Phase 2 |
| AI Pipeline Operations | Operate AI projections | Chunk, vector, graph, embedding migration, freshness | Phase 2 |
| Reconciliation | Trust and data quality | Source/Mongo/chunk/vector/graph count checks | Phase 2 |
| Search Evaluation | AI quality | Golden questions, top-K, citations, lineage correctness | Phase 2/3 |
| Observability | Production operations | Logs, metrics, alerts, failure taxonomy | Yes |
| Security + Audit | Governance | RBAC, production controls, secrets refs, audit | Yes |
| Notifications | Escalation | Failure alerts, SLA misses, regression notifications | Phase 2 |
| Production Governance | Safe prod changes | Approval, reason, diff, rollback | Phase 3 |
| Runbooks + Evidence | Supportability | Runbooks, evidence, UAT, production readiness | Yes |

## Workflow types

- Crawler
- Miner
- Validator
- AI chunk generator
- Vector indexer
- Neo4j graph builder
- Reconciliation job
- Evaluation job
- Utility/maintenance job

## Pipeline examples

### Metadata refresh pipeline

```text
Crawler
→ Canonical Validation
→ AI Chunk Generation
→ Vector Indexing
→ Neo4j Graph Sync
→ Reconciliation
→ Golden Question Smoke Test
```

### Lineage refresh pipeline

```text
Lineage Miner
→ DataFlow/DataProcess Validation
→ Graph Edge Projection
→ Lineage Reconciliation
→ Lineage Golden Questions
```

### Full AI reindex pipeline

```text
AI Chunk Generation Full Rebuild
→ Embedding Generation
→ Vector Index Upsert
→ Semantic Search Smoke Test
→ Publish Index Freshness
```

## MVP capability set

MVP should include:

- Dashboard
- Workflow catalog
- Workflow detail
- Template-backed workflow creation
- Config editor and validation
- Version history
- Manual runs
- Basic schedules
- Run history
- Run detail
- Logs links/viewer
- Retry/cancel
- Audit
- One execution adapter
- Initial crawler/miner templates

Defer until later:

- Drag-and-drop pipeline designer
- Full template UI editing
- Advanced approval workflow
- Automated remediation
- Complex dependency scheduling
- Advanced search quality analytics
