# 07. UI/UX Design and User Journeys

## Purpose

This page defines the recommended UI/UX for the production-grade Data Compass AI Operations Control Plane.

The UI should look like a product-grade metadata operations console, not like a Kubernetes dashboard.

## Navigation

```text
Data Compass AI Operations
├── Dashboard
├── Workflows
├── Pipelines
├── Runs
├── Failures
├── Logs
├── Reconciliation
├── Search Evaluation
├── Templates
├── Connections
├── Schedules
├── Audit
└── Settings
```

## MVP navigation

Start with:

- Dashboard
- Workflows
- Runs
- Failures
- Logs
- Audit

Add in Phase 2:

- Pipelines
- Reconciliation
- Search Evaluation

## Dashboard wireframe

```text
+------------------------------------------------------------------------------------------------+
| Data Compass AI Operations Dashboard                                                            |
+------------------------------------------------------------------------------------------------+
| [Active Workflows] [Running] [Failed 24h] [Stale Pipelines] [Vector Freshness] [Graph Freshness] |
+------------------------------------------------------------------------------------------------+
| AI Pipeline Health                                                                               |
| Snowflake PROD: Crawler ✓ | Validation ✓ | Chunks ✓ | Vector ✓ | Graph ✗ | Eval Not Run         |
| Databricks PROD: Crawler ✓ | Validation ✓ | Chunks ✓ | Vector ✓ | Graph ✓ | Eval ✓              |
+------------------------------------------------------------------------------------------------+
| Recent Failures                                                                                  |
| Workflow/Pipeline        Env   Failed Stage        Error Code              Action               |
| Snowflake Metadata Flow  PROD  Neo4j Graph Sync    GRAPH_EDGE_MISMATCH     View / Retry          |
+------------------------------------------------------------------------------------------------+
```

## Workflow catalog

Columns:

- Name
- Type
- Platform
- Environment
- Status
- Schedule
- Last Run
- Last Run Status
- Next Run
- Owner
- Actions

Actions:

- View
- Run Now
- Edit Configuration
- Enable / Disable
- Clone
- View Runs
- View Logs
- Archive

Filters:

- Platform
- Type
- Environment
- Status
- Last Run Status
- Owner
- Schedule Type
- Search

## Workflow detail

Header:

```text
Databricks PROD Metadata Crawler
Crawler | Databricks | PROD | Active | Current Version v7 | Owner: Data Compass Platform
[Run Now] [Edit Config] [Disable] [Clone] [Audit]
```

Tabs:

- Overview
- Configuration
- Schedule
- Runs
- Logs
- Versions
- Audit

Overview should show description, owner, platform, environment, connection, schedule, resource profile, last run summary, seven-run health, output counts, and failure trend.

## Create workflow wizard

Steps:

1. Select Template
2. Basic Details
3. Source Configuration
4. Connection
5. Schedule
6. Runtime Resources
7. Validate
8. Review and Create

Template cards should show workflow type, platform, required connection, expected outputs, and supported schedule types.

## Configuration editor

Use schema-driven forms generated from template JSON schema.

Modes:

| Mode | User |
|---|---|
| Basic | Operators and workflow admins |
| Advanced | Platform admins |
| Raw JSON | Expert users only, permission-gated |

Validation UX:

- Inline validation.
- Validate button.
- Connection test button.
- Dry run option.
- Scope estimate where possible.
- Warnings for broad source patterns.

## Run Now modal

Fields:

- Workflow version
- Run mode: Full, Incremental, Dry Run, Limited Scope
- Runtime overrides
- Log level
- Reason/comment
- Production warning

For production workflows:

- Require reason.
- Show audit warning.
- Optionally require approval for critical workflows.

## Run detail

Tabs:

- Summary
- Steps
- Logs
- Metrics
- Errors
- Output
- Config Snapshot
- Audit

Step timeline example:

```text
✓ Initialize
✓ Validate Connection
✓ Extract Catalogs
✗ Extract Tables
- Transform to Canonical Model
- Load MongoDB
- Emit Summary
```

## Pipeline detail

DAG view:

```text
[Snowflake Crawler]
        ↓
[Canonical Validation]
        ↓
[AI Chunk Generation]
        ↓
[Vector Indexing]      [Neo4j Graph Projection]
        ↓                    ↓
        [Reconciliation]
              ↓
[Golden Question Smoke Test]
```

Each node displays workflow name, last status, duration, output count, error count, logs link, and retry option.

## Reconciliation screen

Shows source counts, MongoDB counts, AI chunk counts, vector index counts, Neo4j node/edge counts, mismatch counts, orphan relationships, missing JRN counts, and trend over time.

## Search evaluation screen

Shows question set, last run, pass rate, Top-1/Top-3/Top-5 accuracy, citation coverage, lineage correctness, restricted retrieval tests, failed questions, and before/after comparison.

## UX rules

1. Always show environment clearly; PROD should stand out.
2. Always show workflow version used for a run.
3. Always show run mode and runtime overrides.
4. Always show source log links.
5. Do not expose secrets.
6. Show failure stage and recommended next action.
7. Avoid requiring users to understand Kubernetes or Airflow.
8. Keep basic mode simple; advanced mode permission-gated.
