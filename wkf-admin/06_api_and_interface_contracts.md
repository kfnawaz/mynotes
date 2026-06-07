# 06. API and Interface Contracts

## Purpose

This page defines the primary APIs and service-to-service contracts for the control plane.

## API principles

1. APIs enforce permissions; UI is not the security boundary.
2. APIs return normalized statuses, not raw orchestrator-specific statuses.
3. APIs return stable IDs and version references.
4. APIs support pagination, filtering, sorting, and search.
5. APIs produce audit events for mutating actions.
6. APIs return actionable errors with error code, message, retryability, and recommended action.

## API groups

```text
/workflow-templates
/workflows
/workflow-versions
/runs
/pipeline-templates
/pipelines
/pipeline-runs
/schedules
/config-validation
/logs
/metrics
/reconciliation
/evaluation
/audit
/approvals
/connections
/notifications
```

## Key APIs

| API | Method | Purpose |
|---|---|---|
| `/workflow-templates` | GET | List templates available to the user. |
| `/workflow-templates/{templateId}` | GET | Return template schema, defaults, overrides, outputs. |
| `/workflows` | GET | List workflows with filters and pagination. |
| `/workflows` | POST | Create workflow from template. |
| `/workflows/{workflowId}` | GET | Return workflow detail. |
| `/workflows/{workflowId}` | PUT | Update workflow and create new version. |
| `/workflows/{workflowId}/runs` | POST | Trigger manual workflow run. |
| `/runs` | GET | Search/list workflow runs. |
| `/runs/{runId}` | GET | Run detail with steps, summary, logs, config snapshot. |
| `/runs/{runId}/retry` | POST | Retry failed run or step. |
| `/runs/{runId}/cancel` | POST | Cancel queued/running run. |
| `/pipelines` | GET/POST | List/create pipelines. |
| `/pipelines/{pipelineId}` | GET/PUT | View/update pipeline. |
| `/pipelines/{pipelineId}/runs` | POST | Trigger pipeline run. |
| `/pipeline-runs/{pipelineRunId}` | GET | Pipeline run detail. |
| `/config-validation/workflow` | POST | Validate config before save/run. |
| `/runs/{runId}/logs` | GET | Return logs or log references. |
| `/reconciliation/runs` | POST | Trigger reconciliation workflow. |
| `/evaluation/runs` | POST | Run golden-question evaluation. |
| `/audit/events` | GET | Search audit history. |

## Manual run request

```json
{
  "workflowVersion": 7,
  "runMode": "INCREMENTAL",
  "runtimeOverrides": {
    "catalogs": ["risk"],
    "schemas": ["capital"],
    "logLevel": "DEBUG"
  },
  "reason": "Manual validation after configuration change."
}
```

## Manual run response

```json
{
  "runId": "run_20260607_001",
  "status": "QUEUED"
}
```

## Config validation response

```json
{
  "valid": false,
  "errors": [
    {
      "field": "catalogs",
      "code": "REQUIRED_FIELD_MISSING",
      "message": "At least one catalog must be selected."
    }
  ],
  "warnings": [
    {
      "field": "schemas",
      "code": "BROAD_SCOPE_WARNING",
      "message": "Wildcard schema crawl may run longer than usual."
    }
  ]
}
```

## Orchestration adapter interface

```text
submitWorkflowRun(SubmitRunRequest) -> SubmitRunResponse
getWorkflowRunStatus(orchestratorRunId) -> NormalizedRunStatus
getWorkflowRunSteps(orchestratorRunId) -> StepStatus[]
cancelWorkflowRun(orchestratorRunId) -> CancelResult
getLogReference(orchestratorRunId) -> LogReference
retryWorkflowRun(orchestratorRunId, RetryOptions) -> RetryResult
```

## Standard error response

```json
{
  "errorCode": "WORKFLOW_CONFIG_INVALID",
  "message": "Workflow configuration failed validation.",
  "details": [
    {"field": "catalogs", "message": "Catalogs cannot be empty."}
  ],
  "retryable": false,
  "recommendedAction": "Update configuration and validate again.",
  "correlationId": "corr_abc123"
}
```

## API design notes

- Every mutating API must emit audit.
- Every production mutating API must support reason/comment.
- Runtime overrides must be validated against template allow-list.
- Log APIs must enforce log-view permissions.
- Workflow and pipeline APIs should return available actions based on user permission.
