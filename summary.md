Recommendation: Do not choose Option B as written. The BFF should not read MongoDB directly.

Choose Option B+.

The Control Plane should have a dedicated backend Workflow Management API service that owns MongoDB access. That service can read the Harvester’s runSummaries collection and normalize/copy the data into the Control Plane run store for UI, audit, retry, and operational history.

Recommended pattern:

Control Plane SPA
  → Fastify BFF / auth layer
  → Workflow Management API
  → MongoDB

For run history:

Metadata Harvester
  → writes run summary / run events to MongoDB runSummaries or agreed collection
  → Workflow Management API reads/imports runSummaries
  → Control Plane stores normalized workflow_runs / workflow_run_steps / audit records
  → UI reads run history only through Workflow Management API

Final decision:

Choose Option B+:
- BFF remains auth/config/proxy only.
- No MongoDB driver in the BFF.
- No MongoDB access from the SPA.
- Dedicated CP backend API owns MongoDB reads/writes.
- Harvester runSummaries can be the acquisition source.
- CP run store becomes the UI-facing operational history.

Option C can remain a fallback for environments where MongoDB runSummary access is not allowed, but it is more orchestration-heavy and should not be the primary pilot path.