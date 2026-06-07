Create contract test cases

Before building more UI, define tests that prove the harvester is control-plane-ready.

Minimum tests:

validate-only succeeds with valid config
validate-only fails with invalid config
dry-run does not write to MongoDB
full run writes/upserts metadata
structured logs include runId/workflowId/workflowRevision
run summary is produced
failure produces standard error category/code
SIGTERM produces cancelled status
rerun does not create duplicate metadata

These tests should be owned jointly by the harvester and control plane teams.