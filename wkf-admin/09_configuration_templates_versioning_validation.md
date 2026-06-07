# 09. Configuration, Templates, Versioning, and Validation

## Purpose

This page defines how workflow and pipeline configuration should be modeled, edited, validated, versioned, approved, and rolled back.

## Configuration principles

1. Users create workflows from approved templates.
2. Templates define safe configuration schemas.
3. Configurations are edited through schema-driven UI forms.
4. Every saved change creates a new immutable version.
5. Every run references an exact version and runtime overrides.
6. Production changes require a reason and may require approval.
7. Secrets are stored as references, never raw values.

## Template-driven design

A workflow template defines workflow type, platform, container image, entrypoint, default arguments, config schema, defaults, allowed runtime overrides, resource profile defaults, validation rules, expected outputs, and supported orchestration engines.

## Example template schema

```json
{
  "type": "object",
  "required": ["workspace", "catalogs", "targetMongoDatabase"],
  "properties": {
    "workspace": {"type": "string", "title": "Databricks Workspace"},
    "catalogs": {"type": "array", "title": "Catalogs", "items": {"type": "string"}, "minItems": 1},
    "schemas": {"type": "array", "title": "Schemas", "items": {"type": "string"}, "default": ["*"]},
    "includeTables": {"type": "boolean", "default": true},
    "includeViews": {"type": "boolean", "default": true},
    "includeColumns": {"type": "boolean", "default": true},
    "excludePatterns": {"type": "array", "items": {"type": "string"}},
    "targetMongoDatabase": {"type": "string"}
  }
}
```

## Validation levels

| Level | Description | Example |
|---|---|---|
| Schema validation | Field types and required fields | catalogs cannot be empty |
| Business validation | Domain-specific rules | production workflows require owner team |
| Security validation | Permission and environment checks | user cannot modify PROD workflow |
| Connection validation | Connection reference exists and is usable | Databricks token/cert valid |
| Scope validation | Source filters are reasonable | wildcard PROD crawl warning |
| Resource validation | CPU/memory/timeout within allowed profile | timeout cannot exceed policy |
| Output validation | Target output entities are allowed | crawler cannot write event entities |

## Versioning model

Create a new workflow version when config, schedule, resource profile, image, entrypoint, connection reference, or runtime defaults change.

Version record must include:

- workflowId
- version number
- config snapshot
- schedule snapshot
- resource profile snapshot
- image snapshot
- connection refs snapshot
- changed by
- changed at
- change reason
- approval status

## Runtime overrides

Runtime overrides allow safe manual variation without changing saved workflow configuration.

Examples:

- Run only one catalog/schema/table.
- Change log level to DEBUG.
- Use dry-run mode.
- Set lookback window for lineage miner.
- Limit max records for test run.

Runtime overrides must be defined by template, validated before run, stored on the run record, and visible in the config snapshot tab.

## Approval workflow

MVP can capture approval fields without full approval automation. Phase 2/3 should add formal approval for production changes.

Approval required for:

- Production workflow config changes.
- Production schedule changes.
- Production connection reference changes.
- Production resource profile increases.
- Production manual full crawls, if policy requires.
- Disabling critical workflows.

## Rollback

Rollback creates a new version copied from an old version. It should not delete history.

```text
Current version: v8
Rollback target: v6
New version created: v9, copied from v6
Reason: v8 caused failures in production
```

## Pitfalls and mitigations

| Pitfall | Mitigation |
|---|---|
| Hardcoded UI per connector | Use JSON schema-driven forms. |
| Config drift | Version every config and schedule change. |
| Unknown failed-run config | Store config snapshot on every run. |
| Production mistakes | Require reason and approval for critical changes. |
| Secret leakage | Store only connection refs and mask secret-related fields. |
| Overly broad crawls | Add scope warnings and dry-run estimates. |
