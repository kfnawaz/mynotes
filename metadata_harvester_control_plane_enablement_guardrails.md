# Copilot Instructions: Enable Data Compass Metadata Harvester for Control Plane Integration

## Purpose

You are helping enable the **Data Compass Metadata Harvester** so it can be managed by an independent **Data Compass Control Plane**.

The Metadata Harvester already performs crawler and miner work, including metadata extraction, lineage mining, transformation, and loading into the Data Compass MongoDB metadata database.

The goal of this work is **not** to build the control plane inside the Metadata Harvester. The goal is to make the Metadata Harvester **control-plane-ready** by ensuring it can be started, observed, validated, retried, and operated consistently by an external workflow control plane.

---

## Core Design Principle

Keep a clean separation of responsibilities.

### The Control Plane Owns

- Workflow definition
- Workflow configuration management
- Scheduling
- Manual/ad hoc triggering
- Workflow versioning
- Run history
- Run status aggregation
- UI screens
- RBAC and permissions
- Audit history
- Retry/cancel orchestration
- Operator-facing dashboards

### The Metadata Harvester Owns

- Actual crawler and miner execution
- Source platform connectivity
- Metadata extraction
- Lineage mining
- Transformation into the Data Compass metadata model
- MongoDB loading
- Structured execution reporting
- Runtime metrics
- Standard failure reporting
- Safe termination behavior

Do not move control-plane responsibilities into the Metadata Harvester.

The Metadata Harvester should behave like a well-structured, observable, containerized execution worker.

---

## Scope of This Enablement Work

Focus only on changes that make the Metadata Harvester easier to run and observe from a future control plane.

### In Scope

- Standard runtime behavior
- Consistent execution modes
- Structured logging
- Step-level progress reporting
- Runtime metrics
- Run summary output
- Standard failure classification
- Idempotent MongoDB writes
- Deterministic identity handling
- Configuration validation
- Container readiness
- Secret-safe behavior
- Graceful cancellation
- Connector capability documentation
- Contract-oriented tests

### Out of Scope

Do **not** build the following inside the Metadata Harvester:

- Control plane UI
- Workflow administration screens
- Scheduling engine
- Airflow/Argo/Kubernetes orchestration logic as business logic
- Workflow version database
- Control plane audit store
- RBAC authorization engine
- AI chunk generation orchestration
- Vector indexing orchestration
- Neo4j graph orchestration
- GraphRAG orchestration
- Business-facing search or chat UI

---

## Expected Harvester Behavior

The Metadata Harvester should be able to run any supported crawler or miner in a predictable way when invoked by an external orchestrator.

At a high level, each run should be able to:

1. Accept runtime context from the caller.
2. Accept a validated configuration input.
3. Validate the configuration and connection.
4. Execute the requested crawler or miner mode.
5. Emit structured logs and progress information.
6. Write metadata and lineage to MongoDB safely.
7. Produce a clear run summary.
8. Exit with a meaningful status.

The control plane should be able to understand what happened without knowing the internal code structure of the crawler or miner.

---

## Required Runtime Capabilities

Ensure the Metadata Harvester supports a consistent runtime pattern across all crawlers and miners.

The runtime should support:

- A unique run identifier supplied by the external caller
- A workflow identifier supplied by the external caller
- A workflow version or configuration version supplied by the external caller
- A platform or connector identifier
- A run mode
- A configuration input
- A log level
- A target environment

Avoid hardwiring the harvester to one scheduler or execution platform.

The same harvester behavior should work whether the caller is:

- Kubernetes Job
- Kubernetes CronJob
- Argo Workflow
- Airflow DAG
- Local developer command
- Future Data Compass Control Plane

---

## Supported Run Modes

The Metadata Harvester should support clear run modes. Not every connector needs every mode on day one, but the code should move toward a consistent model.

Recommended modes:

- **Validate Only** — validate configuration, connection, permissions, and target availability without extracting or writing data.
- **Dry Run** — execute extraction and transformation logic without writing final results to MongoDB.
- **Full Run** — perform a complete crawl or mining run.
- **Incremental Run** — process only changes since a prior checkpoint, where supported.
- **Limited Scope Run** — run against a restricted catalog, schema, table, report, workbook, query window, or equivalent source-specific scope.
- **Retry Failed Scope** — reprocess failed portions of a previous run, where supported.

Minimum MVP target:

- Validate Only
- Dry Run
- Full Run
- Limited Scope Run

---

## Standard Execution Stages

Use consistent stage names so the control plane can show a useful progress timeline.

For crawler workflows, use stages similar to:

- Initialize
- Validate Configuration
- Validate Connection
- Discover Scope
- Extract Metadata
- Transform to Canonical Model
- Validate Output
- Load to MongoDB
- Emit Summary
- Cleanup

For miner workflows, use stages similar to:

- Initialize
- Validate Configuration
- Validate Connection
- Extract History
- Mine Lineage
- Build DataFlow Records
- Build DataProcess Records
- Validate Lineage
- Load to MongoDB
- Emit Summary
- Cleanup

Do not let every connector invent unrelated stage names unless there is a strong reason. Consistency matters more than perfect source-specific terminology.

---

## Structured Logging Guardrails

The Metadata Harvester must emit structured logs that can be consumed by a control plane or centralized logging platform.

Each log entry should include, at minimum:

- Timestamp
- Severity level
- Application name
- Connector or platform
- Run identifier
- Workflow identifier, when provided
- Workflow/configuration version, when provided
- Stage name
- Human-readable message
- Error code, when applicable
- Retryable indicator, when applicable
- Key metrics, when applicable

Avoid plain text-only logs for important operational events.

Avoid logging secrets, tokens, passwords, private keys, or raw connection strings.

---

## Progress and Event Reporting

The harvester should report meaningful execution progress.

At minimum, it should indicate:

- Run started
- Stage started
- Stage completed
- Stage failed
- Run completed
- Run failed
- Run cancelled
- Summary emitted

This does not require building a complex eventing platform inside the harvester. For MVP, structured logs may be enough. The important point is that the emitted information should be machine-readable and consistent.

---

## Run Summary Requirements

Every run should produce a run summary that can be used by a future control plane run-detail page.

The summary should include:

- Run identifier
- Workflow identifier, if supplied
- Connector/platform
- Run mode
- Start time
- End time
- Duration
- Final status
- Records/assets discovered
- Records/assets created
- Records/assets updated
- Records/assets deprecated
- Records skipped
- Warning count
- Error count
- Collections or entity types touched
- Failure details, if applicable

For miners, also include lineage-specific metrics such as:

- Queries analyzed
- History records read
- Lineage edges created
- DataFlow records created or updated
- DataProcess records created or updated
- Events created, if applicable
- Lineage parse failures

---

## Failure and Error Handling Guardrails

The Metadata Harvester should report failures in a way operators can understand.

Standardize error categories such as:

- Invalid configuration
- Missing connection reference
- Authentication failure
- Source connection failure
- Source permission failure
- Source timeout
- Source rate limit
- Extraction failure
- Transformation failure
- Canonical validation failure
- MongoDB connection failure
- MongoDB write failure
- Lineage parse failure
- Partial load completed
- Unknown failure

Each failure should clearly indicate:

- What failed
- Which stage failed
- Whether the failure is retryable
- What the likely operator action is, where possible

Avoid surfacing only raw stack traces as the primary failure information. Stack traces may be logged, but the run summary should include a clear classified failure.

---

## Exit Behavior

The harvester should exit with meaningful status behavior so an external orchestrator can determine the result.

The implementation should distinguish:

- Success
- Failure
- Partial success
- Timeout
- Cancelled
- Validation failure
- Invalid configuration
- Authentication failure
- MongoDB load failure

Do not allow all failures to collapse into a generic error without classification.

---

## Idempotency and Safe Reruns

The control plane will eventually allow retries, manual reruns, and scheduled reruns. Therefore, the Metadata Harvester must be safe to run repeatedly.

Ensure:

- MongoDB writes use upsert behavior where appropriate.
- Stable resource identity is preserved across runs.
- JRN generation is deterministic and stable.
- Re-running the same source scope does not create duplicate assets.
- Records can be stamped with the run that last harvested or updated them.
- Removed source assets are handled through a controlled deprecation strategy.
- Partial failures do not corrupt future reruns.

This is one of the most important guardrails. Do not treat reruns as one-time bulk inserts.

---

## Data Compass Metadata Model Alignment

The harvester should continue to emit data aligned to the Data Compass metadata model.

Ensure crawler and miner outputs preserve:

- Source identity
- Stable JRN values
- Entity type
- Lifecycle status
- Ownership where available
- Node references where available
- Relationship references
- Classification fields where available
- Source system and source object identifiers
- Last harvested timestamp
- Last seen timestamp
- Run identifier that produced or updated the record

The control plane and downstream AI architecture depend on clean, stable metadata. Do not sacrifice identity and relationship quality for faster loading.

---

## Configuration Guardrails

The Metadata Harvester should treat configuration as an external runtime input.

Do:

- Validate configuration before execution.
- Fail fast on invalid configuration.
- Provide clear validation errors.
- Support connector-specific configuration schemas or documented config expectations.
- Keep source scope, target settings, runtime settings, and connector options logically separated.
- Support limited-scope overrides for debugging.

Do not:

- Hardcode production source scopes in code.
- Require code changes for simple configuration changes.
- Read workflow definitions directly from the control plane database.
- Store secrets in configuration files.
- Log raw configuration if it may contain sensitive values.

---

## Secret Handling Guardrails

The Metadata Harvester should never require raw secrets to be stored in workflow definitions or logs.

Use secret references and approved runtime secret resolution mechanisms.

Acceptable patterns include:

- Kubernetes secret mounts
- Environment variables injected by the runtime
- Vault or enterprise secret provider lookup
- Mounted credential files

Always mask secrets in:

- Logs
- Errors
- Run summaries
- Debug output
- Validation responses

---

## Container and Deployment Readiness

The Metadata Harvester should be packaged so it can run cleanly as an external job.

Ensure:

- Versioned container image is available.
- Entrypoint is clear.
- Runtime arguments are documented.
- Configuration input path is supported.
- Health or validation command is available.
- Connector version is visible.
- Application version is visible.
- Logs are written to stdout/stderr in structured form.
- Runtime does not require manual shell interaction.

The same container should be runnable locally for development and in Kubernetes or another orchestrator for production.

---

## Cancellation and Shutdown

The harvester should handle graceful cancellation.

When receiving a termination signal, it should:

- Stop starting new source reads.
- Safely stop or finish the current batch.
- Avoid corrupting partially written records.
- Emit a cancellation message or event.
- Emit a partial summary where possible.
- Exit in a way the orchestrator can classify as cancelled.

This is needed because the control plane will eventually expose a Cancel Run action.

---

## Incremental Processing and Checkpointing

Where source platforms support it, the harvester should support incremental operation.

For incremental-capable connectors, define how checkpoints are stored and used.

Checkpointing should support:

- Last successful run
- Last processed timestamp
- Last processed source cursor or ID, where applicable
- Lookback windows for lineage miners
- Restart after failure
- Retry of failed scopes

Do not make checkpointing a control-plane-only concern. The harvester owns source-specific incremental logic.

---

## Metrics Guardrails

The harvester should emit useful operational metrics.

Common metrics should include:

- Records read
- Records written
- Records skipped
- Records failed
- Assets discovered
- Assets created
- Assets updated
- Assets deprecated
- Warnings count
- Errors count
- Duration
- Source API call count
- Source API failure count
- MongoDB write count
- MongoDB write failure count

Connector-specific metrics should be added where useful.

Examples:

- Databricks: catalogs, schemas, tables, views, columns processed
- Snowflake miner: query history rows, access history rows, queries parsed, lineage edges created
- Tableau: sites, projects, workbooks, dashboards, datasources, fields processed
- Alteryx: workflows, tools, inputs, outputs, lineage links processed

---

## Connector Capabilities Documentation

Each connector should document what it supports.

At minimum, document:

- Connector name
- Platform
- Crawler or miner type
- Supported run modes
- Required configuration fields
- Optional configuration fields
- Supported source scopes
- Expected output entity types
- Supported stages
- Known limitations
- Retry behavior
- Incremental support
- Dry-run support
- Validation support

This documentation allows the control plane to create accurate workflow templates later.

---

## Shared Runtime Utility Recommendation

Avoid implementing runtime behavior separately in every connector.

Create or enhance a shared Metadata Harvester runtime utility that handles:

- Runtime argument parsing
- Config loading
- Config validation
- Structured logging
- Stage tracking
- Metrics capture
- Summary generation
- Secret masking
- Error classification
- Exit status mapping
- Signal handling

Connector code should focus on source-specific extraction, mining, transformation, and loading.

---

## Testing Expectations

Add tests that prove the harvester is control-plane-ready.

Recommended tests:

- Valid config passes validation.
- Invalid config fails with a clear error.
- Validate-only mode does not write to MongoDB.
- Dry-run mode does not write final records to MongoDB.
- Full run uses idempotent upsert behavior.
- Structured logs include run and stage context.
- Run summary is produced.
- Failure is classified with a retryable indicator where applicable.
- Secrets are masked in logs and summaries.
- Cancellation is handled gracefully.
- Rerun does not create duplicate records.

Add connector-specific tests where needed.

---

## Documentation to Add or Update

Add or update documentation in the Metadata Harvester repository.

Recommended documents:

- Control plane readiness overview
- Runtime invocation guide
- Configuration guide
- Connector capability matrix
- Supported run modes
- Logging and metrics guide
- Error taxonomy
- Idempotency and MongoDB write behavior
- Secret handling guide
- Local run examples
- Container run examples

This documentation should be written for both developers and platform operators.

---

## Development Guardrails

When making changes, follow these guardrails:

1. Do not tightly couple the Metadata Harvester to a specific control plane implementation.
2. Do not introduce UI or workflow scheduling logic into the harvester.
3. Do not make the harvester read control-plane-owned workflow definitions directly.
4. Do not log secrets or raw credentials.
5. Do not create duplicate metadata records on rerun.
6. Do not bypass canonical model validation.
7. Do not make connector behavior inconsistent unless the source platform truly requires it.
8. Do not hide failures behind generic exceptions.
9. Do not add one-off behavior that only works for one scheduler.
10. Keep the harvester runnable locally, in Kubernetes, and from future orchestration tools.

---

## Immediate Implementation Priority

Start with the smallest useful control-plane-readiness slice:

1. Standardize runtime invocation.
2. Standardize runtime configuration input.
3. Add structured JSON logs.
4. Add stage-level progress reporting.
5. Add run summary generation.
6. Add standard failure classification.
7. Ensure idempotent MongoDB writes.
8. Add validate-only and dry-run modes.
9. Add connector capability documentation.
10. Add tests for the above behavior.

Do this for one high-priority crawler first, then apply the same pattern to miners and other connectors.

Recommended first connector:

- Databricks metadata crawler or Snowflake metadata crawler

After one connector proves the pattern, generalize the runtime utility and roll it out across the rest of the Metadata Harvester connectors.

---

## Definition of Done

The Metadata Harvester is considered control-plane-ready when an external system can:

- Start a crawler or miner run with runtime context and config.
- Know which workflow and run the execution belongs to.
- Track what stage the run is in.
- View structured logs by run and stage.
- Understand whether the run succeeded, failed, partially succeeded, timed out, or was cancelled.
- See meaningful metrics and output counts.
- See a clear run summary.
- Retry or rerun safely without duplicate metadata records.
- Trust that secrets are not exposed.
- Use documented connector capabilities to create workflow templates.

The harvester should not need to know which UI, scheduler, or control plane is calling it.

