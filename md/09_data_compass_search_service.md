# 09. Data Compass Search Service

## 1. Purpose

The Data Compass Search Service provides controlled APIs for exact metadata lookup, semantic vector search, graph traversal, hybrid search, source hydration, and source citation output.

This service is the retrieval boundary for AI agents and UI clients. Agents should not directly query MongoDB, vector indexes, or Neo4j without going through controlled APIs.

## 2. Role in Architecture

```
Data Compass UI / Agent Layer
   -> Search Service
      -> Exact Metadata Search against MongoDB
      -> Semantic Search against Vector Index
      -> Graph Search against Neo4j
      -> Source Hydration from MongoDB
   -> GraphRAG Orchestrator
```

## 3. Design Principles

| Principle | Description |
| --- | --- |
| Controlled access | Central API enforces filters and auditing. |
| Multiple retrieval modes | Exact, semantic, graph, and hybrid search are supported. |
| Source-backed results | Every result includes source guid/jrn and source collection. |
| Security first | Entitlement, lifecycle, and classification filters apply before results are returned. |
| Deduplication | Results are deduplicated by source jrn. |
| Explainability | Results should include match reason, chunk type, score, and source reference. |

## 4. Search Modes

| Mode | Best For | Example |
| --- | --- | --- |
| Exact Search | Known identifiers, table names, jrn, guid, owner, schema | Find table customer_transactions. |
| Semantic Search | Business-language discovery | Find assets related to customer risk. |
| Graph Search | Lineage and dependency traversal | What is downstream of table X? |
| Hybrid Search | Mixed intent questions | Find customer risk datasets and show related reports. |
| Source Hydration | Retrieve authoritative records | Hydrate result sources from MongoDB. |

## 5. API Capabilities

### Exact Metadata Search API

Inputs:

- query text
- entity types
- exact identifiers
- filters
- pagination

Outputs:

- source guid
- source jrn
- entity type
- display name
- lifecycle status
- matched fields
- source collection

### Semantic Search API

Inputs:

- natural-language query
- entity filters
- chunk type filters
- metadata filters
- topK

Outputs:

- chunk_id
- source guid/jrn
- chunk text/snippet
- score
- chunk type
- metadata
- relationships

### Graph Search API

Inputs:

- starting jrn
- traversal direction
- max depth
- relationship filters
- entity filters

Outputs:

- graph paths
- nodes
- relationships
- source references
- path depth

### Hybrid Search API

Combines:

- exact matching
- semantic vector matching
- graph expansion
- deduplication
- ranking
- source hydration

## 6. Request Envelope

```json
{
  "requestId": "",
  "userContext": {
    "userId": "",
    "roles": [],
    "allowedNodes": [],
    "allowedClassifications": [],
    "businessUnit": ""
  },
  "query": {
    "text": "find datasets related to customer risk",
    "mode": "semantic | exact | graph | hybrid",
    "entityTypes": ["Dataset", "Distribution"],
    "chunkTypes": ["asset_summary"],
    "topK": 10
  },
  "filters": {
    "lifecycleStatus": ["PUBLISHED", "APPROVED"],
    "databaseType": "DATABRICKS",
    "nodeId": "",
    "owner": "",
    "confidentiality": []
  }
}
```

## 7. Response Envelope

```json
{
  "requestId": "",
  "modeUsed": "hybrid",
  "results": [
    {
      "rank": 1,
      "score": 0.91,
      "matchType": "semantic",
      "matchReason": "Matched customer risk description in asset summary chunk.",
      "source": {
        "guid": "",
        "jrn": "",
        "collection": "Distribution",
        "entityType": "Distribution",
        "lifecycleStatus": "PUBLISHED"
      },
      "snippet": "",
      "metadata": {},
      "relationships": {}
    }
  ],
  "appliedFilters": {},
  "warnings": []
}
```

## 8. Ranking Strategy

Ranking should consider:

- semantic similarity score
- exact match strength
- lifecycle status
- certification/approval status where available
- owner/user affinity where allowed
- graph proximity
- source freshness
- chunk type priority
- classification/access eligibility

## 9. Deduplication Rules

- Deduplicate by source_jrn where possible.
- Keep the best scoring chunk per source unless multiple chunk types are intentionally requested.
- Preserve multiple graph paths when they explain different lineage routes.
- Do not merge results across different entity types unless they share a canonical identity.

## 10. Source Hydration

Source hydration fetches authoritative MongoDB source records for selected results.

Rules:

- Hydration must use source guid or jrn.
- Hydration must apply access filters.
- Hydration should return only fields needed by downstream answer generation.
- Hydration failure must be visible in warnings.

## 11. Security Filters

Search Service must enforce:

- lifecycleStatus filtering
- user role filtering
- owner/node filtering
- confidentiality filtering
- PII/SPI/MNPI handling
- environment/source platform scoping
- restricted metadata exclusion before LLM context assembly

## 12. Observability

Log:

- requestId
- userId or service identity
- query mode
- input query
- filters applied
- result counts
- source jrns returned
- latency by backend
- failures/warnings

## 13. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| SRCH-001 | Build exact search API | P0 |
| SRCH-002 | Build semantic search API | P0 |
| SRCH-003 | Apply security filters to search | P0 |
| SRCH-004 | Return source guid/jrn and matched fields | P0 |
| SRCH-005 | Build graph search API | P1 |
| SRCH-006 | Build hybrid search API | P1 |
| SRCH-007 | Build source hydration | P1 |
| SRCH-008 | Build source citation output | P1 |
| SRCH-009 | Build ranking and deduplication | P1 |
| SRCH-010 | Add observability and audit logging | P1 |

## 14. Acceptance Criteria

- Exact search returns assets by guid, jrn, name, schema, and owner.
- Semantic search returns ranked chunks with source references.
- Graph search returns upstream/downstream paths by jrn.
- Hybrid search combines and deduplicates results.
- All search APIs apply security and lifecycle filters.
- Results include enough metadata for answer grounding and UI display.

## 15. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-060 | Build Exact Metadata Search API |
| DC-AI-061 | Build Semantic Search API |
| DC-AI-062 | Build Graph Search API |
| DC-AI-063 | Build Hybrid Search API |
| DC-AI-064 | Build Source Record Hydration |
| DC-AI-065 | Build Source Citation Output |
| DC-AI-066 | Build Result Deduplication |
| DC-AI-067 | Build Result Ranking Rules |

## 16. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-SRCH-001 | What latency target is required for each search mode? | API and index design. |
| OQ-SRCH-002 | Which fields can be returned in snippets for restricted assets? | Security and UX. |
| OQ-SRCH-003 | Should search support saved filters and bookmarked assets in MVP? | Context Plane integration. |
