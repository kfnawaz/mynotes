# 07. Embedding and Vector Indexing

## 1. Purpose

The Embedding and Vector Indexing capability converts AI-ready chunks into vector embeddings and stores them in a searchable index with strong metadata filters. This provides semantic discovery over Data Compass metadata while preserving source traceability, lifecycle filtering, classification filtering, and reindexing controls.

## 2. Role in Architecture

```
AI Chunk Builder
   -> Embedding Service Wrapper
   -> Vector Upsert Service
   -> MongoDB Atlas Vector Index
   -> Semantic Search API
   -> Hybrid Search / GraphRAG
```

## 3. Recommended Initial Platform

MongoDB Atlas Vector Search is the recommended initial vector platform because:

- Data Compass metadata is already MongoDB-centered.
- Chunk documents can live close to source metadata.
- Metadata filtering is critical for lifecycle, owner, node, and classification rules.
- It reduces the number of moving parts for MVP.
- Neo4j remains available for graph traversal while MongoDB handles document/vector retrieval.

## 4. Design Principles

| Principle | Description |
| --- | --- |
| Store chunks, not raw records | Only AI-ready chunks should be embedded. |
| Preserve traceability | Every vector record must include source guid and jrn. |
| Filter before context | Metadata filters must be applied before results are used by LLM. |
| Deterministic upsert | Use chunk_id to update, not duplicate. |
| Version everything | Track chunk schema, template, embedding model, and embedding timestamp. |
| Rebuildable | Vector index can be deleted and rebuilt from canonical metadata and chunks. |

## 5. In Scope

- Embedding model selection and configuration.
- Embedding wrapper service.
- Vector index definition.
- Vector record schema.
- Metadata filter design.
- Upsert behavior.
- Delete/deprecate behavior.
- Reindexing by source jrn, entity type, and embedding model version.
- Semantic search smoke tests.
- Cost and latency tracking.

## 6. Out of Scope

- Creating chunk text.
- Graph traversal.
- Final answer generation.
- Updating authoritative metadata.
- UI result rendering.

## 7. Vector Record Schema

```json
{
  "_id": "<chunk_id>",
  "chunk_id": "<chunk_id>",
  "text": "<AI-ready chunk text>",
  "embedding": [0.012, -0.044],
  "source": {
    "guid": "",
    "jrn": "",
    "collection": "Distribution",
    "entity_type": "Distribution",
    "version": 1,
    "lifecycleStatus": "PUBLISHED"
  },
  "metadata": {
    "chunk_type": "asset_summary",
    "nodeId": "",
    "nodeType": "",
    "owners": [],
    "ownerRoles": [],
    "databaseType": "DATABRICKS",
    "entityStoreName": "",
    "entitySchemaName": "",
    "entityPhysicalType": "TABLE",
    "confidentiality": "INTERNAL",
    "hasPII": false,
    "hasSPI": false,
    "hasMNPI": false,
    "isDeprecated": false,
    "sourcePlatform": "Databricks"
  },
  "relationships": {
    "datasetJrn": "",
    "upstreamJrns": [],
    "downstreamJrns": [],
    "dataServiceJrns": [],
    "reportJrns": []
  },
  "embeddingInfo": {
    "model": "",
    "dimensions": 0,
    "embeddingVersion": 1,
    "chunkSchemaVersion": 1,
    "templateVersion": 1,
    "embeddedAt": ""
  }
}
```

## 8. Metadata Filters

The vector index must support filtering by:

| Filter | Purpose |
| --- | --- |
| entity_type | Limit search to Distribution, Dataset, Attribute, etc. |
| chunk_type | Search summaries, attributes, lineage, classifications separately. |
| lifecycleStatus | Exclude DRAFT/REJECTED/DEPRECATED by default. |
| nodeId/nodeType | Enforce node-based access and scoping. |
| owners/ownerRoles | Support owner-based filtering. |
| databaseType | Limit to Databricks, Snowflake, etc. |
| entityStoreName/entitySchemaName | Catalog/database/schema filtering. |
| confidentiality | Control restricted metadata. |
| hasPII/hasSPI/hasMNPI | Security and governance filters. |
| sourcePlatform | Source-specific search. |

## 9. Embedding Model Selection Criteria

| Criterion | Consideration |
| --- | --- |
| Accuracy | Must retrieve relevant assets for business-language questions. |
| Cost | Must be acceptable for expected chunk volume and reindexing frequency. |
| Latency | Embedding generation should support batch and ad hoc reindexing. |
| Dimensions | Must align with vector index configuration. |
| Security | Must comply with enterprise data handling rules. |
| Versioning | Model version must be stored on every vector record. |

## 10. Embedding Flow

```
1. Read pending chunks from ai_metadata_chunks.
2. Validate chunk text and metadata.
3. Generate embedding through embedding service wrapper.
4. Attach embedding model metadata.
5. Upsert vector record by deterministic chunk_id.
6. Update indexing status.
7. Log success/failure counts.
```

## 11. Upsert Rules

- Upsert by chunk_id.
- Existing vector record with same chunk_id must be replaced.
- Source guid/jrn cannot be changed without generating a new chunk identity.
- EmbeddingInfo must be updated on every successful embedding.
- Failure must not delete existing good vector record unless explicitly configured.

## 12. Delete and Deprecate Rules

| Source State | Vector Behavior |
| --- | --- |
| Source deleted | Delete or mark inactive based on retention policy. |
| lifecycleStatus = DEPRECATED | Exclude by default; keep for historical search if enabled. |
| lifecycleStatus = REJECTED | Remove or inactive. |
| lifecycleStatus = DRAFT | Exclude unless restricted draft search is enabled. |
| Classification becomes restricted | Re-embed if text changed; update metadata filters immediately. |

## 13. Reindexing Scenarios

| Scenario | Required Action |
| --- | --- |
| Chunk template changed | Regenerate and re-embed affected chunk types. |
| Embedding model changed | Re-embed all active chunks or selected entity types. |
| Source record changed | Regenerate chunks for source jrn and related rollups. |
| Relationship changed | Regenerate dependent chunks and graph projection. |
| Classification changed | Update metadata and re-check security filters. |
| Lifecycle changed | Update active/deprecated filtering state. |

## 14. Semantic Search Smoke Test

Initial smoke test should include at least 20 questions across:

- Business-language discovery.
- Technical table/schema lookup.
- Attribute discovery.
- Classification/gov discovery.
- Service exposure.
- Lineage-related discovery.

Example questions:

1. Find datasets related to customer risk.
1. Which assets contain customer identifier fields?
1. Find Databricks tables in schema capital.
1. Which distributions are related to liquidity reporting?
1. Find metadata for trade exposure.

## 15. Quality Metrics

| Metric | MVP Target |
| --- | --- |
| Top-5 semantic retrieval relevance | 80%+ on golden questions. |
| Vector upsert success rate | 95%+ for valid chunks. |
| Duplicate chunk rate | 0% for deterministic chunk IDs. |
| Deprecated asset leakage | 0 by default. |
| Source traceability coverage | 100% active vector records. |
| Metadata filter coverage | 100% active vector records. |

## 16. Operational Logging

Log:

- chunk_id
- source_guid
- source_jrn
- entity_type
- chunk_type
- embedding model
- embedding status
- error message
- retry count
- elapsed time
- token/character count
- vector upsert status

## 17. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| VEC-001 | Select embedding model and dimensions | P0 |
| VEC-002 | Define vector record schema | P0 |
| VEC-003 | Create MongoDB Atlas vector index | P0 |
| VEC-004 | Build embedding service wrapper | P0 |
| VEC-005 | Build vector upsert service | P0 |
| VEC-006 | Apply metadata filters in search | P0 |
| VEC-007 | Build delete/deprecate handling | P1 |
| VEC-008 | Track embedding version | P1 |
| VEC-009 | Build reindex by source jrn | P1 |
| VEC-010 | Build reindex by entity type | P1 |
| VEC-011 | Build semantic smoke tests | P1 |

## 18. Acceptance Criteria

- Vector index is created with correct dimensions.
- Chunks can be embedded and upserted by deterministic chunk_id.
- Search results include source guid, jrn, entity type, chunk type, and lifecycle status.
- Metadata filters work for lifecycle, entity type, node, owner, and classification fields.
- Deprecated records are excluded by default.
- Reindexing a single source jrn replaces old vector records.
- Smoke test search produces reviewed results for golden questions.

## 19. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-040 | Select Embedding Model and Configuration |
| DC-AI-041 | Build Embedding Service Wrapper |
| DC-AI-042 | Create MongoDB Atlas Vector Index |
| DC-AI-043 | Build Vector Upsert Service |
| DC-AI-044 | Build Vector Delete/Deprecate Handling |
| DC-AI-045 | Build Semantic Search Smoke Test |
| DC-AI-046 | Track Embedding Model Version |
| DC-AI-047 | Build Reindex by Source JRN |
| DC-AI-048 | Build Reindex by Entity Type |

## 20. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-VEC-001 | Which embedding model is approved for enterprise use? | Blocks implementation. |
| OQ-VEC-002 | What is the expected chunk volume by entity type? | Cost and sizing. |
| OQ-VEC-003 | Should deprecated chunks be retained but filtered or physically deleted? | Reindexing and audit. |
| OQ-VEC-004 | What latency target is required for semantic search? | Index/query design. |
