The workflow ID: is it going to be a UUID or a slug or both? 
Thought for 9s

Use both, but do not treat them as the same thing.

My recommendation:

workflowId  = immutable system ID, UUID or ULID
workflowKey = human-readable slug

Example:

{
  "workflowId": "01HZV7A9Q6K2B8Q3QW8N5M4P2X",
  "workflowKey": "databricks-prod-metadata-crawler",
  "name": "Databricks PROD Metadata Crawler",
  "environment": "PROD",
  "platform": "DATABRICKS",
  "type": "CRAWLER",
  "currentRevision": 7
}
Best choice

Use:

Primary internal identity: ULID or UUID
Human/operator identity: slug

I slightly prefer ULID over UUID because it is sortable by creation time and still globally unique. UUID is also fine.

Why not only slug?

A slug is readable:

databricks-prod-metadata-crawler

But it can become problematic if:

name changes
environment changes
team wants to rename it
two teams create similar workflows
workflow is cloned
old run history must remain stable

If the slug is your only ID, renames become dangerous.

Why not only UUID/ULID?

A UUID/ULID is stable, but not friendly:

01HZV7A9Q6K2B8Q3QW8N5M4P2X

Operators do not want to troubleshoot with only that.

They want to say:

“The Snowflake PROD lineage miner failed.”

So you need a readable key too.

Recommended naming

Use these fields:

{
  "workflowId": "01HZV7A9Q6K2B8Q3QW8N5M4P2X",
  "workflowKey": "snowflake-prod-lineage-miner",
  "displayName": "Snowflake PROD Lineage Miner"
}
Field meanings
Field	Purpose	Mutable?
workflowId	Permanent system identity	No
workflowKey	Human-readable unique key / slug	Prefer no, but can support controlled rename
displayName	UI name	Yes
workflowRevision	Config version number	Increments
configHash	Exact snapshot fingerprint	No
Recommended run record

Every run should store both the ID and key snapshot.

{
  "runId": "run_01HZV7F4S6Z7K9",
  "workflowId": "01HZV7A9Q6K2B8Q3QW8N5M4P2X",
  "workflowKey": "snowflake-prod-lineage-miner",
  "workflowName": "Snowflake PROD Lineage Miner",
  "workflowRevision": 7,
  "configHash": "sha256:abc123",
  "status": "SUCCEEDED"
}

This way, even if the workflow is renamed later, old runs still show the name/key used at execution time.

Slug rules

Generate slugs like:

{platform}-{environment}-{purpose}

Examples:

databricks-prod-metadata-crawler
databricks-uat-metadata-crawler
snowflake-prod-lineage-miner
tableau-prod-metadata-crawler
alteryx-dev-lineage-miner

Use uniqueness constraints like:

tenantId + workflowKey must be unique

or, if there is no tenant:

environment + workflowKey must be unique
Final recommendation

Use this pattern:

workflowId       = immutable ULID/UUID, used by APIs, DB references, runs, audit
workflowKey      = readable slug, used in UI, search, logs, operator conversations
displayName      = friendly name, freely editable
workflowRevision = integer config version
configHash       = exact config snapshot fingerprint

That gives you stable engineering identity and clean operational usability.



What's a better choice for workflow version: an integer or a semver or a hash? 
Thought for 8s

For workflow version, the best choice is:

Use a monotonically increasing integer revision as the primary workflow version, and also store a content hash for immutability/integrity.

Do not use SemVer as the main workflow version.

Best model

Use this:

{
  "workflowId": "wf_databricks_prod_crawler",
  "workflowRevision": 7,
  "workflowVersionId": "wf_databricks_prod_crawler:v7",
  "configHash": "sha256:abc123...",
  "templateVersion": "1.3.0",
  "harvesterVersion": "2.1.4",
  "containerImage": "data-compass/metadata-harvester:2.1.4",
  "containerDigest": "sha256:def456..."
}
Why integer is best for workflow version

A workflow version represents a configuration snapshot, not a software release.

Example changes:

v1: Databricks crawler created
v2: Added finance catalog
v3: Changed schedule from 2 AM to 3 AM
v4: Increased timeout
v5: Changed schema filter
v6: Disabled lineage option
v7: Updated resource profile

These are simple revisions. A human-readable integer is perfect.

Use:

workflowRevision = 1, 2, 3, 4, 5...

This makes troubleshooting easy:

“Run failed on workflow revision 7, but revision 6 was successful.”

That is exactly what operators need.

Why not SemVer for workflow config?

SemVer is better for software artifacts, not operational configuration.

SemVer means:

MAJOR.MINOR.PATCH

That implies semantic compatibility rules:

1.0.0 → 1.0.1 = patch
1.0.0 → 1.1.0 = minor enhancement
1.0.0 → 2.0.0 = breaking change

But for workflow config, it becomes awkward:

Is changing a cron schedule a patch?
Is changing a catalog filter a minor version?
Is changing timeout a patch?
Is changing source scope a major version?

The team will waste time debating version numbers.

Use SemVer for:

Harvester application version
Connector version
Workflow template version
Runtime contract version
SDK version

Example:

{
  "templateVersion": "1.2.0",
  "harvesterVersion": "2.0.1",
  "runtimeContractVersion": "1.0.0"
}
Why not hash as the main workflow version?

A hash is excellent for integrity, but bad for humans.

Example:

sha256:9f86d081884c7d659a2feaa0c55ad015...

That is not useful in meetings or operations.

Nobody wants to say:

“The failure happened on config hash 9f86d081...”

But hash is very useful behind the scenes because it proves the exact config used.

Use hash for:

immutability
change detection
deduplication
audit
proving exact config snapshot
detecting accidental mutation
Recommended final design
1. Workflow definition

Current editable workflow.

{
  "workflowId": "wf_snowflake_lineage_miner_prod",
  "name": "Snowflake PROD Lineage Miner",
  "currentRevision": 12,
  "status": "ACTIVE"
}
2. Workflow version snapshot

Immutable config snapshot.

{
  "workflowId": "wf_snowflake_lineage_miner_prod",
  "revision": 12,
  "versionId": "wf_snowflake_lineage_miner_prod:v12",
  "configSnapshot": {},
  "scheduleSnapshot": {},
  "resourceProfileSnapshot": {},
  "configHash": "sha256:abc123...",
  "createdBy": "sid123",
  "createdAt": "2026-06-08T10:00:00Z",
  "changeReason": "Expanded query history lookback from 7 days to 14 days"
}
3. Workflow run

Every run points to the exact revision used.

{
  "runId": "run_20260608_001",
  "workflowId": "wf_snowflake_lineage_miner_prod",
  "workflowRevision": 12,
  "workflowVersionId": "wf_snowflake_lineage_miner_prod:v12",
  "configHash": "sha256:abc123...",
  "status": "SUCCEEDED"
}
Versioning rules I recommend
Item	Version Type	Example
Workflow config snapshot	Integer revision	revision: 7
Workflow version ID	Human-readable composite	wf_x:v7
Config integrity	SHA-256 hash	sha256:abc123
Workflow template	SemVer	1.4.0
Metadata Harvester app	SemVer	2.1.0
Connector implementation	SemVer	databricks-crawler:1.8.2
Runtime contract	SemVer	1.0.0
Container identity	Image tag + digest	2.1.0, sha256:def456
Final recommendation

Use this combination:

Primary workflow version = integer revision
Integrity/version fingerprint = hash
Software/template/contract version = SemVer

So the clean answer is:

Integer for workflow version, hash for exact snapshot integrity, SemVer for harvester/template/runtime software versions.

That gives you human-friendly operations, strong auditability, and clean separation between configuration changes and software releases.