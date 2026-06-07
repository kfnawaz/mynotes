# 14. Operations, Monitoring, and Reindexing

## 1. Purpose

This page defines the operational capabilities required to run, monitor, troubleshoot, and reindex the Data Compass AI Modernization platform.

Without strong operations, users will not trust AI search results. Every derived projection must be observable, explainable, and rebuildable.

## 2. Operational Principles

| Principle | Description |
| --- | --- |
| Source reconciliation | Compare MongoDB source records to chunks, vector records, and graph nodes. |
| Reindexability | Rebuild one asset, one entity type, or entire projections. |
| Failure visibility | Failed records must be visible with reason and retry status. |
| Idempotent jobs | Re-running jobs should not create duplicates. |
| Auditability | Search, indexing, and answer sources must be traceable. |
| Cost awareness | Embedding and query costs should be tracked. |

## 3. Indexing Status Model

```json
{
  "source_guid": "",
  "source_jrn": "",
  "entity_type": "Distribution",
  "source_version": 1,
  "source_updatedTimestamp": "",
  "chunk_status": "pending | success | failed | skipped",
  "vector_status": "pending | success | failed | skipped",
  "graph_status": "pending | success | failed | skipped",
  "last_indexed_at": "",
  "last_graph_synced_at": "",
  "failure_reason": "",
  "retry_count": 0
}
```

## 4. Dashboards

### Indexing Dashboard

Show:

- records by entity type
- chunk generation status
- vector indexing status
- graph sync status
- failed records
- retry counts
- last successful run

### Reconciliation Dashboard

Show:

- MongoDB source count
- chunk count
- vector record count
- Neo4j node count
- Neo4j relationship count
- mismatch count

### Search Quality Dashboard

Show:

- golden question pass rate
- failed search cases
- top-K retrieval metrics
- no-answer count
- user feedback trend

## 5. Reindexing Modes

| Mode | Purpose |
| --- | --- |
| Reindex by source jrn | Fix one asset. |
| Reindex by guid | Fix one source record. |
| Reindex by entity type | Rebuild all Distributions, Datasets, etc. |
| Reindex changed since timestamp | Incremental catch-up. |
| Reindex by chunk type | Rebuild asset_summary, attribute_summary, etc. |
| Reindex by template version | Rebuild after template change. |
| Reindex by embedding model version | Re-embed after model change. |
| Rebuild graph projection | Full graph rebuild. |

## 6. Failure Queue

Failure records should include:

- source guid
- source jrn
- entity type
- stage failed
- error code
- error message
- retryable flag
- retry count
- last attempted timestamp
- owner/team

## 7. Common Failure Types

| Failure | Likely Cause | Handling |
| --- | --- | --- |
| Missing jrn | Source quality issue | Quarantine and report. |
| Missing relationship target | Orphan reference | Log and continue partial projection. |
| Invalid lifecycle | Data validation issue | Mark validation failed. |
| Embedding API failure | Transient service/rate limit | Retry with backoff. |
| Vector upsert failure | Index/config issue | Retry after fix. |
| Neo4j constraint failure | Duplicate or bad key | Investigate identity rules. |
| Access filter failure | Missing security metadata | Block from search until resolved. |

## 8. Runbooks

Required runbooks:

- How to reindex one asset by jrn.
- How to reindex one entity type.
- How to rebuild vector index.
- How to rebuild Neo4j graph.
- How to investigate failed chunks.
- How to investigate low search quality.
- How to investigate missing lineage.
- How to handle embedding model change.
- How to handle schema/template version change.

## 9. Monitoring Metrics

| Metric | Purpose |
| --- | --- |
| ingestion records processed | Throughput. |
| validation failure rate | Source quality. |
| chunk generation success rate | Chunk builder health. |
| embedding success rate | Embedding health. |
| vector upsert latency | Vector performance. |
| graph load success rate | Graph health. |
| search latency | User experience. |
| GraphRAG answer latency | User experience. |
| retrieval quality score | Search quality. |
| feedback negative rate | User trust. |

## 10. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| OPS-001 | Build indexing dashboard | P0 |
| OPS-002 | Build failed indexing queue | P0 |
| OPS-003 | Build reindex by jrn | P1 |
| OPS-004 | Build reindex by entity type | P1 |
| OPS-005 | Build vector count reconciliation | P1 |
| OPS-006 | Build graph count reconciliation | P1 |
| OPS-007 | Build search quality dashboard | P1 |
| OPS-008 | Create operational runbooks | P1 |
| OPS-009 | Add cost and latency monitoring | P2 |

## 11. Acceptance Criteria

- Operators can see indexing state by entity type.
- Failed records are visible with reason and retry count.
- One asset can be reindexed by jrn.
- Entity type reindexing is supported.
- Source/chunk/vector/graph counts can be reconciled.
- Runbooks exist for common support scenarios.

## 12. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-110 | Build Indexing Dashboard |
| DC-AI-111 | Build Failed Indexing Queue |
| DC-AI-112 | Build Reindex by JRN |
| DC-AI-113 | Build Reindex by Entity Type |
| DC-AI-114 | Build Vector Count Reconciliation |
| DC-AI-115 | Build Graph Count Reconciliation |
| DC-AI-116 | Build Search Quality Dashboard |
| DC-AI-117 | Create Operational Runbooks |
| DC-AI-118 | Add Cost and Latency Monitoring |

## 13. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-OPS-001 | What tool will host the operational dashboard? | Implementation. |
| OQ-OPS-002 | What alerting thresholds are required? | Support model. |
| OQ-OPS-003 | Who owns failed record remediation? | Operating model. |
