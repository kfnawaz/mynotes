2. Error category vocabulary

Define a canonical controlled vocabulary based on the “Failure and Error Handling Guardrails” requirements.

Do not let each crawler/miner invent its own error categories.

Recommended approach:

Use a fixed enum of error categories and stable error codes.
Allow connector-specific details in the error message/details field, not in the category itself.

Suggested categories:

CONFIGURATION
AUTHENTICATION
AUTHORIZATION
SOURCE_CONNECTION
SOURCE_API
SOURCE_RATE_LIMIT
EXTRACTION
TRANSFORMATION
CANONICAL_VALIDATION
MONGODB_LOAD
LINEAGE_MINING
TIMEOUT
CANCELLATION
PARTIAL_SUCCESS
UNKNOWN

Example:

{
  "category": "SOURCE_CONNECTION",
  "errorCode": "DATABRICKS_CONNECTION_TIMEOUT",
  "message": "Timed out connecting to Databricks workspace",
  "retryable": true
}
3. Cancellation grace period

Kubernetes default 30s is probably too tight for crawler/miner jobs that may need to flush an in-flight batch.

Recommended default:

terminationGracePeriodSeconds = 120

Make it configurable per workflow:

default: 120s
minimum: 30s
maximum: 300s

Harvester behavior on SIGTERM:

1. Stop accepting new work.
2. Stop pulling new source data.
3. Finish or safely stop current batch.
4. Flush partial summary/checkpoint.
5. Emit RUN_CANCELLED event.
6. Exit cleanly before SIGKILL.

Also, the harvester should checkpoint/flush periodically so it does not depend entirely on a long shutdown window.

4. Run summary delivery

Use both stdout and a known file path, but make the control plane the owner of the final stored run summary.

Recommended pattern:

Harvester emits SUMMARY_EMITTED event to stdout as structured JSON.
Harvester also writes final summary JSON to a known path.
Control plane/orchestrator reads the summary and stores it in the control plane run store.
UI reads run summaries from the control plane API, not directly from harvester output.

Recommended path:

/var/run/data-compass/run-summary.json

This gives you flexibility:

stdout = easy log-based capture
known path = reliable artifact capture
control plane run store = authoritative UI source
5. Capability manifest delivery

For MVP, this is acceptable:

python scripts/list_capabilities.py --json

But the better production pattern is:

Bake a static capability manifest into the image
AND
provide a standard command to print it.

Recommended command:

metadata-harvester manifest --json

Recommended behavior:

The command reads the baked manifest file and prints it as JSON.

Example baked path:

/app/manifest/capabilities.json

Final recommendation:

MVP: scripts/list_capabilities.py --json is acceptable.
Production: metadata-harvester manifest --json backed by a static manifest inside the image.

This keeps the manifest stable, versioned with the image, and easy for the control plane to discover.