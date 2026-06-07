Data Compass Metadata Harvester team/app to produce a document called:

Data Compass Metadata Harvester — Control Plane Integration Contract

The purpose of this document is to clearly state:

1. What the control plane can expect from the harvester
2. What the harvester expects from the control plane
3. What the harvester will provide
4. What the harvester will not own or provide
5. What evidence proves the harvester is ready

You can give them this prompt.

# Request: Data Compass Metadata Harvester Control Plane Integration Contract

Please create a document that describes how the Data Compass Metadata Harvester is prepared for integration with the independent Data Compass Control Plane.

The document should focus on boundaries, responsibilities, expectations, and readiness evidence. Do not make the harvester dependent on the control plane implementation. The harvester should remain an independently runnable execution application.

## 1. Purpose

Explain the purpose of the integration contract.

The control plane will manage workflow definitions, schedules, configuration versions, manual triggers, run history, logs, audit, and operational visibility.

The Metadata Harvester will execute crawler and miner logic, load metadata/lineage into MongoDB, and emit standard execution evidence.

## 2. Responsibility Boundary

Clearly define the separation of responsibilities.

### Control Plane owns

- Workflow creation and management
- Workflow configuration UI
- Configuration versioning
- Scheduling
- Manual/ad hoc run trigger
- Run history
- Run status tracking
- Audit events
- RBAC and permissions
- Operator UI
- Retry/cancel orchestration
- Future AI pipeline orchestration placeholders

### Metadata Harvester owns

- Metadata crawling
- Lineage mining
- Source system connectivity
- Config validation at runtime
- Extraction and transformation
- Canonical metadata output
- MongoDB load/upsert behavior
- Structured logs
- Step-level progress reporting
- Run summary generation
- Standard exit codes
- Error categorization
- Secret masking
- Graceful cancellation behavior
- Idempotent execution

## 3. What the Harvester Expects from the Control Plane

List the information the control plane must provide when starting a harvester run.

At minimum, describe these categories:

- Run identity
- Workflow identity
- Workflow revision/version
- Runtime configuration
- Run mode
- Environment
- Connection or secret reference
- Runtime overrides, if any
- Logging level
- Output target details
- Cancellation signal through orchestrator, when applicable

The document should not require the harvester to read directly from the control plane database.

Preferred pattern:

Control Plane resolves workflow configuration and passes runtime inputs to the harvester at execution time.

## 4. What the Harvester Provides to the Control Plane

Describe what the harvester will provide back to the control plane or orchestrator.

At minimum:

- Capability manifest
- Config schema per connector
- Validate-only execution mode
- Dry-run execution mode
- Full execution mode
- Limited-scope execution mode, where supported
- Structured JSON logs
- Standard step names
- Step progress events or equivalent structured logs
- Final run summary
- Metrics
- Standard exit codes
- Error category and error code
- Retryability indicator
- Version information
- Container image/version information
- Documentation and sample configs

## 5. What the Harvester Does Not Provide

Explicitly state what the harvester will not own.

The harvester should not provide or own:

- UI screens
- Workflow scheduling
- Workflow definition storage
- Workflow version management
- RBAC decisions
- Approval workflow
- Audit ownership
- Control plane run history
- Pipeline orchestration
- Retry/cancel policy ownership beyond graceful handling
- Business-facing AI search
- GraphRAG orchestration
- Vector indexing orchestration
- Neo4j orchestration
- Long-term log storage
- Secret management UI

The harvester should only support these through clean execution behavior and emitted evidence.

## 6. Supported Workflow Types

List which crawler and miner types are supported.

Examples:

- Databricks metadata crawler
- Snowflake metadata crawler
- Tableau metadata crawler
- Alteryx metadata crawler
- Databricks lineage miner
- Snowflake lineage miner
- Tableau dependency miner
- Alteryx lineage miner

For each type, document:

- Connector name
- Platform
- Workflow type: crawler or miner
- Supported run modes
- Expected output entities
- Known limitations

## 7. Supported Run Modes

Document the run modes supported by the harvester.

Recommended modes:

- VALIDATE_ONLY
- DRY_RUN
- FULL
- INCREMENTAL, if supported
- LIMITED_SCOPE, if supported
- RETRY_FAILED, if supported

For each mode, explain:

- What it does
- Whether it writes to MongoDB
- What output/evidence it produces
- Expected use by the control plane

## 8. Logging and Observability Expectations

Document how the harvester emits logs and operational evidence.

Include:

- Structured log format
- Required common fields
- Step names
- Metrics
- Failure information
- Secret masking expectations
- Sample successful log
- Sample failed log

The control plane should be able to associate every log line with:

- runId
- workflowId
- workflowRevision
- platform
- connector
- step
- timestamp
- severity

## 9. Run Summary Expectations

Document the final run summary produced by the harvester.

The summary should include:

- Run ID
- Workflow ID
- Workflow revision
- Status
- Start time
- End time
- Duration
- Assets discovered
- Assets created
- Assets updated
- Assets deprecated
- Records read/written/skipped/failed
- Lineage edges created, for miners
- Warnings
- Errors
- Collections touched
- Failure details, if applicable

Clarify whether the summary is written to stdout, a known path, or both.

## 10. Error Handling Expectations

Document the error category vocabulary and exit code behavior.

The document should include:

- Error categories
- Error codes
- Retryable vs non-retryable errors
- Standard exit codes
- Example authentication failure
- Example invalid config failure
- Example MongoDB load failure
- Example partial-success case

## 11. Idempotency Expectations

Document how the harvester avoids duplicate metadata on reruns or retries.

Include:

- Deterministic JRN behavior
- Upsert behavior
- runId stamping
- lastHarvestedAt / lastSeenAt behavior
- Soft deprecation behavior for removed source assets
- Duplicate prevention expectations

## 12. Secret Handling Expectations

Document how secrets are handled.

The harvester should:

- Accept secret references, not raw secrets
- Resolve secrets through approved runtime mechanisms
- Mask secrets in logs
- Avoid secrets in summaries
- Avoid secrets in error messages
- Avoid secrets in generated artifacts

## 13. Cancellation Expectations

Document how the harvester handles cancellation.

Include:

- SIGTERM/SIGINT handling
- Stopping new work
- Completing or safely stopping current batch
- Writing partial summary
- Emitting cancellation evidence
- Exit code behavior
- Recommended termination grace period

## 14. Compatibility and Versioning

Document the version information the harvester exposes.

Include:

- Harvester application version
- Connector version
- Runtime contract version
- Config schema version
- Container image tag
- Container image digest
- Capability manifest version

## 15. Readiness Evidence

Provide a readiness checklist proving the harvester is ready for control plane integration.

At minimum:

- Manifest command works
- Config schemas exist
- Sample configs exist
- Validate-only mode works
- Dry-run mode works
- Full run works
- Structured logs are emitted
- Step progress is visible
- Run summary is generated
- Standard exit codes are used
- Error taxonomy is followed
- Secrets are masked
- Cancellation is handled
- MongoDB writes are idempotent
- Re-running does not create duplicates
- Container image is versioned
- Documentation exists
- Contract tests pass

## 16. Integration Assumptions

Document any assumptions the harvester makes about:

- Runtime environment
- Kubernetes or container execution
- Mounted config files
- Environment variables
- Secret access
- MongoDB connectivity
- Network access to source systems
- Logging platform
- File system paths for artifacts

## 17. Open Questions

List any unresolved questions that need agreement between the harvester and control plane teams.

Examples:

- Should run summary be written to stdout, file path, or both?
- Should the harvester post events to a Run Event API, or only emit structured logs?
- What is the default cancellation grace period?
- Which run modes are mandatory for MVP?
- Which connectors support incremental mode?
- Which connectors support limited-scope mode?
- What is the final error category vocabulary?

The key point is this:

The harvester should publish an integration contract, not absorb control plane responsibilities.

That contract becomes the handshake between both systems. It lets the control plane evolve independently while the Metadata Harvester remains a clean, containerized execution engine.