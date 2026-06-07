# 00. Program Index — Data Compass AI Operations Control Plane

## Purpose

This package defines a production-grade control plane for operating the full Data Compass AI architecture. It starts from the current crawler/miner capability and expands into a governed operational platform for workflows, pipelines, AI indexes, graph projection, reconciliation, evaluation, and audit.

## Recommended reading order

1. Executive Summary and Program Vision
2. Production-Grade Target Architecture
3. Control Plane Scope and Capability Model
4. Component Architecture and Responsibilities
5. Data Model and State Management
6. API and Interface Contracts
7. UI/UX Design and User Journeys
8. Workflow and Pipeline Orchestration Design
9. Configuration, Templates, Versioning, and Validation
10. Logging, Metrics, and Observability
11. Security, RBAC, Audit, and Governance
12. AI Pipeline Operations, Reconciliation, and Evaluation
13. Implementation Phases and Sequencing
14. Operating Model, Runbooks, and Production Readiness
15. Risks, Pitfalls, ADRs, and Open Questions
16. Jira Epics Overview
17. Detailed Jira Stories

## Control-plane concept

```text
Data Compass AI Operations Control Plane
├── Product / UX layer
│   ├── Dashboard
│   ├── Workflow Administration
│   ├── Pipeline Operations
│   ├── Runs and Logs
│   ├── Failures
│   ├── Reconciliation
│   ├── Search Evaluation
│   └── Audit
├── Control-plane services
│   ├── Workflow Management Service
│   ├── Pipeline Orchestration Service
│   ├── Template and Configuration Service
│   ├── Run State Service
│   ├── Scheduler Service
│   ├── Orchestration Adapter Service
│   ├── Observability Service
│   ├── Audit Service
│   ├── Notification Service
│   ├── Policy / RBAC Service
│   └── AI Operations Service
├── Execution engines
│   ├── Argo Workflows
│   ├── Kubernetes Jobs / CronJobs
│   └── Airflow DAGs, optional adapter
├── Data Compass AI pipeline jobs
│   ├── Crawlers
│   ├── Miners
│   ├── Canonical validators
│   ├── AI chunk generators
│   ├── Vector indexers
│   ├── Neo4j graph builders
│   ├── Reconciliation jobs
│   └── Evaluation jobs
└── Persistence and observability
    ├── MongoDB control-plane collections
    ├── MongoDB Data Compass metadata collections
    ├── MongoDB vector index / AI chunks
    ├── Neo4j graph projection
    ├── Log platform
    ├── Metrics platform
    └── Audit/event store
```

## Core recommendation

Build this as a **control plane**, not a thin UI over Kubernetes cron jobs or Airflow DAGs.

The UI should never directly manage Kubernetes YAML, Airflow DAG code, or pod specs. It should call controlled backend APIs that validate configuration, enforce permissions, version every change, create run records, and delegate execution to an orchestrator through adapters.
