# 01. Executive Summary and Program Vision

## Executive summary

Data Compass already has connector scripts that can crawl metadata from multiple data platforms and mine lineage information into MongoDB. Those crawlers and miners are the foundation, but a production-grade AI metadata architecture needs a broader operational control layer.

The proposed solution is the **Data Compass AI Operations Control Plane**.

This control plane will allow authorized users and operators to:

- Configure crawler, miner, validator, indexer, graph builder, reconciliation, and evaluation workflows.
- Schedule workflows and pipelines.
- Trigger ad hoc runs safely from a UI.
- Monitor status, steps, logs, output counts, failures, retries, and audit events.
- Manage workflow templates and configuration versions.
- Orchestrate end-to-end AI metadata pipelines, not just individual jobs.
- Rebuild AI chunks, vector indexes, and Neo4j graph projections.
- Run reconciliation between source metadata, MongoDB, vector indexes, and Neo4j.
- Run golden-question evaluations for semantic search and GraphRAG quality.
- Provide production-safe governance through RBAC, approvals, audit, and secrets controls.

## Why this is needed

Running crawler/miner scripts through Kubernetes CronJobs, Airflow DAGs, or Argo Workflows solves execution, but it does not solve operational control.

Without a control plane, the team will struggle with:

| Problem | Why it matters |
|---|---|
| No single operational view | Operators cannot quickly see what ran, failed, or became stale. |
| Configuration drift | Job configs live in YAML, scripts, DAG code, or tribal knowledge. |
| Weak run traceability | Failures cannot easily be tied to exact config version, image, input scope, or actor. |
| Manual support burden | Platform engineers become the bottleneck for routine retries and ad hoc runs. |
| Poor auditability | Production changes and manual runs need evidence. |
| No end-to-end AI pipeline health | A crawler can succeed while vector indexing or graph sync fails later. |
| Low user trust | AI answers become questionable when index freshness, graph sync, or validation status is unknown. |

## Program vision

The control plane should become the **operations cockpit** for the Data Compass AI architecture.

It should answer these questions:

- Which crawlers and miners are configured?
- Which workflows ran recently?
- Which workflows failed and why?
- Which config version was used for a failed run?
- Can I safely rerun or retry the failed stage?
- Did metadata load into MongoDB?
- Did canonical validation pass?
- Were AI chunks generated?
- Were embeddings created?
- Was the vector index updated?
- Was Neo4j synced?
- Do source, MongoDB, vector, and graph counts reconcile?
- Did the golden-question evaluation pass?
- Who changed or triggered the workflow?

## Business outcomes

| Outcome | Description |
|---|---|
| Operational transparency | One place to see crawler, miner, indexing, graph, and evaluation health. |
| Faster recovery | Operators can retry, rerun, or escalate failures with logs and config snapshots. |
| Better governance | Production changes, manual runs, and config updates are controlled and audited. |
| Improved metadata trust | Validation, reconciliation, and evaluation results show whether AI-ready metadata is reliable. |
| Lower platform dependency | Authorized users do not need direct access to Kubernetes, Argo, Airflow, or shell scripts. |
| Faster connector onboarding | New connectors can be added through templates and schema-driven configuration. |
| Scalable AI operations | Vector reindex, graph projection, search evaluation, and reconciliation are managed as first-class workflows. |

## Primary users

| User | Needs |
|---|---|
| Platform Admin | Manage templates, execution backends, RBAC, production controls, and troubleshooting. |
| Workflow Admin | Create workflows, edit configurations, schedules, and resource profiles. |
| Metadata Operator | Monitor runs, inspect logs, retry failures, and review output counts. |
| Data Platform Owner | View and trigger workflows for their platform or source. |
| Data Governance Lead | Review validation, lineage, classification, and reconciliation status. |
| SRE / Operations | Monitor availability, latency, errors, retries, stuck runs, and SLA breaches. |
| Auditor | Review who changed, triggered, approved, cancelled, or retried workflows. |
| AI/Search Product Owner | Review vector freshness, graph sync, golden-question results, and quality trends. |

## Scope boundary

The control plane is for administrators and operators. It should not replace the business-facing Data Compass search/chat experience.

### Include

- Workflow administration
- Pipeline orchestration
- Run monitoring
- Logs and errors
- Configuration and versioning
- Scheduling
- Reindexing
- Graph sync control
- Reconciliation
- Search/GraphRAG evaluation
- RBAC, audit, approval, and operational evidence

### Exclude or link out

- End-user AI chat/search UI
- Business glossary authoring
- Manual catalog curation UI
- Full enterprise log platform replacement
- Full ticketing system replacement
- Free-form Kubernetes/Airflow administration

## Guiding principles

1. **Control-plane first**: UI calls APIs; APIs govern execution.
2. **No direct UI-to-orchestrator coupling**: UI does not talk directly to Kubernetes, Argo, or Airflow.
3. **Templates over custom code**: New workflows are created from approved templates.
4. **Every config is versioned**: Every run is tied to an exact workflow or pipeline version.
5. **Every run is auditable**: Trigger, actor, config, output, logs, and status are traceable.
6. **Metadata freshness is visible**: Users can see whether MongoDB, vector index, and graph are in sync.
7. **Security before execution**: RBAC and policy checks happen before create, edit, run, retry, cancel, or log access.
8. **Derived projections are rebuildable**: Vector and graph stores are operational outputs, not independent sources of truth.
9. **Failure should be actionable**: Errors have stage, code, retryability, likely cause, and recommended next action.
10. **Start simple, then mature**: MVP avoids drag-and-drop DAG editing and focuses on template-driven workflows and pipeline operations.
