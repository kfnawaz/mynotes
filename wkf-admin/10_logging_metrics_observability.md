# 10. Logging, Metrics, and Observability

## Purpose

This page defines the observability model for workflows, pipelines, and AI operations.

## Observability goals

- Every run is traceable from UI to orchestrator to container logs.
- Every log line correlates to workflowId, runId, stepId, platform, and environment.
- Operators see failure stage, error code, retryability, and recommended action.
- Metrics show throughput, latency, records processed, errors, retries, and output counts.
- Dashboards show individual workflow health and end-to-end pipeline health.

## Correlation identifiers

All jobs should emit:

- workflowId
- workflowVersion
- runId
- pipelineId, if applicable
- pipelineRunId, if applicable
- stepId
- sourcePlatform
- environment
- orchestratorRunId
- traceId / correlationId

## Structured log format

```json
{
  "timestamp": "2026-06-07T10:00:00Z",
  "level": "INFO",
  "workflowId": "wf_databricks_prod_crawler",
  "workflowVersion": 7,
  "runId": "run_20260607_001",
  "stepId": "extract_tables",
  "sourcePlatform": "DATABRICKS",
  "environment": "PROD",
  "message": "Extracted table metadata from catalog risk.",
  "metrics": {
    "tablesRead": 812,
    "columnsRead": 12033
  }
}
```

## Log storage strategy

| Data | Store |
|---|---|
| Raw logs | Enterprise log platform |
| Log reference | MongoDB run record |
| Error summary | MongoDB run/step failure fields |
| Error excerpt | MongoDB, limited retention |
| Metrics summary | MongoDB run/step summary and metrics platform |
| Full metrics | Metrics platform |

Do not store full raw logs in MongoDB long-term.

## Metrics to collect

Workflow metrics:

- Run duration
- Queue wait time
- Step duration
- Records read/written/skipped/failed
- Assets created/updated/deprecated
- Lineage edges created
- Warnings and errors
- Retry count
- Memory and CPU usage
- Source API latency
- MongoDB write latency

Pipeline metrics:

- Pipeline duration
- Node duration
- Node success/failure count
- Failed stage
- Reconciliation mismatch count
- Evaluation pass rate
- Freshness SLA result

AI operations metrics:

- Chunks generated
- Embeddings generated
- Vector records upserted/failed
- Graph nodes/edges created/updated
- Golden questions passed/failed
- Top-K retrieval accuracy
- Citation coverage

## Failure taxonomy

| Category | Example error codes |
|---|---|
| Authentication | AUTH_FAILURE, TOKEN_EXPIRED, CERT_INVALID |
| Authorization | PERMISSION_DENIED, SOURCE_ACCESS_DENIED |
| Connectivity | NETWORK_TIMEOUT, DNS_FAILURE, PROXY_ERROR |
| Source API | SOURCE_RATE_LIMIT, SOURCE_API_ERROR, SOURCE_SCHEMA_CHANGED |
| Configuration | CONFIG_INVALID, SCOPE_INVALID, CONNECTION_REF_MISSING |
| Data Quality | REQUIRED_FIELD_MISSING, JRN_MISSING, ORPHAN_RELATIONSHIP |
| MongoDB Load | MONGO_WRITE_FAILED, DUPLICATE_KEY, VALIDATION_ERROR |
| Vector Index | EMBEDDING_FAILED, VECTOR_UPSERT_FAILED, INDEX_NOT_FOUND |
| Graph Sync | NEO4J_WRITE_FAILED, GRAPH_CONSTRAINT_ERROR, EDGE_MISSING |
| Orchestration | JOB_SUBMIT_FAILED, POD_FAILED, DAG_FAILED, TIMEOUT |
| Evaluation | GOLDEN_QUESTION_FAILED, CITATION_MISSING, RESTRICTED_RESULT_RETURNED |

## Alerts

Alert for:

- Critical production workflow failed.
- Pipeline failed at required stage.
- Workflow has not run within freshness SLA.
- Reconciliation mismatch exceeds threshold.
- Search evaluation pass rate drops below threshold.
- Unauthorized retrieval test fails.
- Consecutive failures exceed threshold.
- Run duration exceeds expected threshold.
- Schedule missed.

## Log viewer requirements

- Search within logs.
- Filter by step.
- Filter by severity.
- Show correlated run metadata.
- Copy log line.
- Download logs, if allowed.
- Link to enterprise logging platform.
- Redact sensitive values.
