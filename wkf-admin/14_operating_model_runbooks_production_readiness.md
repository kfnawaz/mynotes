# 14. Operating Model, Runbooks, and Production Readiness

## Purpose

This page defines the operating model and production readiness requirements for the Data Compass AI Operations Control Plane.

## Operating model

| Role | Responsibility |
|---|---|
| Product Owner | Prioritizes features and user experience. |
| Platform Engineering | Owns workflow services, orchestration adapters, deployment, and reliability. |
| Metadata Engineering | Owns crawler/miner templates and metadata output quality. |
| AI/Search Engineering | Owns chunking, vector indexing, graph sync, and evaluation workflows. |
| SRE/Operations | Owns monitoring, alerting, runbooks, and incident response. |
| Security/Governance | Owns RBAC, approvals, audit, and secrets controls. |
| Connector Owners | Own platform-specific connector configs and failure investigation. |

## Support tiers

| Tier | Scope |
|---|---|
| L1 | Check dashboard, failed runs, retry known failures, collect evidence. |
| L2 | Investigate logs, config changes, source access, connector issues. |
| L3 | Code-level debugging, orchestrator issues, platform defects. |

## Standard runbooks

Create runbooks for:

- Workflow failed.
- Pipeline failed at crawler stage.
- Pipeline failed at validation stage.
- Pipeline failed at vector indexing stage.
- Pipeline failed at graph sync stage.
- Reconciliation mismatch.
- Golden-question evaluation failed.
- Manual production run requested.
- Workflow schedule missed.
- Workflow stuck running.
- Logs unavailable.
- Orchestrator submission failed.
- Connection validation failed.

## Runbook template

```markdown
# Runbook: <Failure / Scenario>

## Symptoms

## Impact

## First checks

## Common causes

## Resolution steps

## Retry guidance

## Escalation

## Evidence to capture

## Related dashboards

## Related Jira component
```

## Production readiness checklist

### Architecture

- [ ] Target architecture documented.
- [ ] Control-plane store defined.
- [ ] Execution adapter selected.
- [ ] Workflow and pipeline models separated.
- [ ] Run state model defined.

### Security

- [ ] Authentication integrated.
- [ ] RBAC implemented.
- [ ] Production permissions configured.
- [ ] Secrets stored as references.
- [ ] Audit events captured.
- [ ] Log access permissioned.

### Reliability

- [ ] Services deployable and restart-safe.
- [ ] Run records durable.
- [ ] Job execution idempotency considered.
- [ ] Retry/cancel behavior tested.
- [ ] Schedules resilient to service restarts.

### Observability

- [ ] Run logs correlated by runId.
- [ ] Metrics emitted.
- [ ] Failure codes standardized.
- [ ] Dashboards available.
- [ ] Alerts configured.

### Operations

- [ ] Runbooks created.
- [ ] Support ownership defined.
- [ ] Failed-run triage tested.
- [ ] Reindex/retry tested.
- [ ] UAT users trained.

### Data quality and AI quality

- [ ] Canonical validation workflow available.
- [ ] Reconciliation workflow available.
- [ ] Golden-question smoke test available.
- [ ] Vector/graph freshness visible.

## UAT scenarios

1. Create workflow from template.
2. Edit workflow config and save new version.
3. Trigger manual run.
4. View run details and logs.
5. Retry failed run.
6. Disable workflow schedule.
7. Review audit trail.
8. Run pipeline and view DAG status.
9. Run reconciliation and review mismatch.
10. Run golden-question smoke evaluation.

## Pilot recommendation

Start with:

- 2 source platforms.
- 3 to 5 workflows.
- 1 end-to-end metadata refresh pipeline.
- 1 vector indexing workflow.
- 1 graph sync workflow.
- 1 reconciliation workflow.
- 1 golden-question smoke test.

## Success measures

- Reduction in manual platform intervention.
- Mean time to detect failed workflow.
- Mean time to recover failed workflow.
- Percentage of runs with complete config snapshots.
- Percentage of failures with classified error codes.
- Percentage of workflows managed through templates.
- Reconciliation pass rate.
- Golden-question pass rate.
