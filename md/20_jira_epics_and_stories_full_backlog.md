# 20. Jira Epics and Stories - Full Backlog

## Initiative

**Data Compass AI Modernization**

Deliver an AI-ready Data Compass metadata platform using MongoDB as the authoritative source, AI-ready chunks for vector search, Neo4j for graph traversal, GraphRAG for grounded answers, and a Context Plane for enterprise interpretation and feedback.

## EPIC 1: Architecture and Planning

**Description:** Define the architecture, scope, decisions, risks, and delivery structure.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-001 | Create Confluence Space and Page Structure | Create parent and child pages for the project package. | P0 |
| DC-AI-002 | Document Current-State Architecture | Capture current system design and component responsibilities. | P0 |
| DC-AI-003 | Document Target-State Architecture | Define target architecture, principles, and store responsibilities. | P0 |
| DC-AI-004 | Create Architecture Decision Log | Create ADR page and initial ADR templates. | P0 |
| DC-AI-005 | Define MVP Scope and Phasing | Document MVP entities, deferred scope, and phase plan. | P0 |
| DC-AI-006 | Create Risk and Pitfall Register | Capture risks, mitigations, and owners. | P0 |
| DC-AI-007 | Create Open Questions Register | Capture unresolved design and platform questions. | P0 |
| DC-AI-008 | Create Golden Question Categories | Define initial evaluation question categories. | P1 |

## EPIC 2: Canonical Metadata Foundation

**Description:** Define normalized model, identity rules, relationships, lifecycle, and classification handling.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-010 | Confirm MVP Entity Scope | Confirm in-scope entities for MVP. | P0 |
| DC-AI-011 | Define GUID and JRN Identity Rules | Define how guid and jrn are used across chunks, graph, search, citations. | P0 |
| DC-AI-012 | Define Deterministic Chunk ID Format | Define stable chunk ID pattern for upsert and reindexing. | P0 |
| DC-AI-013 | Define Canonical Field Requirements | Document required/optional fields for MVP entities. | P0 |
| DC-AI-014 | Define Relationship Resolution Rules | Define how *.jrn relationships are resolved. | P0 |
| DC-AI-015 | Define Lifecycle Status Handling | Define indexing/search behavior by lifecycle status. | P0 |
| DC-AI-016 | Define Classification Metadata Handling | Define PII/SPI/confidentiality metadata handling. | P0 |

## EPIC 3: Metadata Ingestion Service

**Description:** Read, validate, resolve, and prepare MongoDB metadata.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-020 | Build MongoDB Reader for MVP Entities | Read selected collections in batch mode. | P0 |
| DC-AI-021 | Build Entity Validator | Validate required fields and log invalid records. | P0 |
| DC-AI-022 | Build JRN Relationship Resolver | Resolve related entities using jrn references. | P0 |
| DC-AI-023 | Build Node Resolver | Resolve node references using nodeId and nodeType. | P0 |
| DC-AI-024 | Build Owner Resolver | Normalize owner SID and role metadata. | P1 |
| DC-AI-025 | Build Indexing Status Collection | Track vector and graph projection status. | P0 |
| DC-AI-026 | Build Source Change Detection | Detect records requiring reindexing. | P1 |
| DC-AI-027 | Build Retry and Failure Handling | Capture failed records and support retry. | P1 |
| DC-AI-028 | Build Ingestion Summary Report | Summarize processed, valid, invalid, skipped records. | P1 |
| DC-AI-029 | Build Ad Hoc Ingestion by JRN | Allow a single record to be reprocessed. | P1 |

## EPIC 4: AI Document and Chunk Builder

**Description:** Convert metadata into deterministic, source-backed, AI-readable chunks.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-030 | Define AI Chunk Schema | Define chunk_id, text, source, metadata, relationships. | P0 |
| DC-AI-031 | Build Distribution Summary Chunk Template | Generate readable summary chunks for Distributions. | P0 |
| DC-AI-032 | Build Dataset Summary Chunk Template | Generate readable summary chunks for Datasets. | P0 |
| DC-AI-033 | Build Attribute-Level Chunk Template | Generate column/attribute chunks. | P0 |
| DC-AI-034 | Build DataService Chunk Template | Generate service exposure/interface chunks. | P1 |
| DC-AI-035 | Build DataFlow Lineage Chunk Template | Generate provider-consumer lineage chunks. | P1 |
| DC-AI-036 | Build DataProcess Transformation Chunk Template | Generate process/transformation chunks. | P1 |
| DC-AI-037 | Build Report Summary Chunk Template | Generate report and dependency chunks. | P1 |
| DC-AI-038 | Build Chunk Generation Batch Job | Generate chunks by entity type or source jrn. | P0 |
| DC-AI-039 | Build Chunk Quality Review Samples | Produce sample chunks for stakeholder review. | P1 |

## EPIC 5: Embedding and Vector Indexing

**Description:** Generate embeddings, store vector records, support filtering and reindexing.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-040 | Select Embedding Model and Configuration | Confirm model, dimensions, cost, security. | P0 |
| DC-AI-041 | Build Embedding Service Wrapper | Generate embeddings with retry/error handling. | P0 |
| DC-AI-042 | Create MongoDB Atlas Vector Index | Configure vector index and metadata filters. | P0 |
| DC-AI-043 | Build Vector Upsert Service | Store embeddings and metadata by chunk_id. | P0 |
| DC-AI-044 | Build Vector Delete/Deprecate Handling | Deactivate/delete chunks for lifecycle changes. | P1 |
| DC-AI-045 | Build Semantic Search Smoke Test | Validate search using golden questions. | P1 |
| DC-AI-046 | Track Embedding Model Version | Store model/schema version per chunk. | P1 |
| DC-AI-047 | Build Reindex by Source JRN | Rebuild vectors for one source asset. | P1 |
| DC-AI-048 | Build Reindex by Entity Type | Rebuild vectors for an entity type. | P1 |

## EPIC 6: Graph Projection and Neo4j Builder

**Description:** Build Neo4j graph projection and traversal support.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-050 | Define Neo4j Graph Schema | Define labels, relationship types, properties, indexes. | P0 |
| DC-AI-051 | Build Cypher Schema Runner | Create repeatable Neo4j schema deployment. | P0 |
| DC-AI-052 | Build Distribution/Dataset Graph Loader | Load asset nodes and IMPLEMENTS relationships. | P0 |
| DC-AI-053 | Build Attribute Graph Loader | Load attributes and CONTAINS relationships. | P0 |
| DC-AI-054 | Build DataService Graph Loader | Load service nodes and EXPOSES relationships. | P1 |
| DC-AI-055 | Build DataFlow Graph Loader | Load observed lineage relationships. | P1 |
| DC-AI-056 | Build DataProcess Graph Loader | Load process consumption/production relationships. | P1 |
| DC-AI-057 | Build Report Dependency Graph Loader | Load report dependency relationships. | P1 |
| DC-AI-058 | Build Upstream Traversal Query | Query upstream dependencies by jrn. | P1 |
| DC-AI-059 | Build Downstream Traversal Query | Query downstream dependencies by jrn. | P1 |

## EPIC 7: Data Compass Search Service

**Description:** Provide exact, semantic, graph, hybrid search APIs.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-060 | Build Exact Metadata Search API | Search by guid, jrn, name, schema, owner, report ID. | P0 |
| DC-AI-061 | Build Semantic Search API | Search AI chunks using natural-language queries. | P0 |
| DC-AI-062 | Build Graph Search API | Expose upstream/downstream and related asset queries. | P1 |
| DC-AI-063 | Build Hybrid Search API | Combine exact, semantic, and graph results. | P1 |
| DC-AI-064 | Build Source Record Hydration | Fetch authoritative MongoDB source records. | P1 |
| DC-AI-065 | Build Source Citation Output | Return source entity, guid, jrn, supporting fields. | P1 |
| DC-AI-066 | Build Result Deduplication | Deduplicate results by source jrn. | P1 |
| DC-AI-067 | Build Result Ranking Rules | Rank by relevance, lifecycle, ownership, graph proximity. | P1 |

## EPIC 8: GraphRAG Orchestration

**Description:** Orchestrate retrieval, graph expansion, context, and answers.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-070 | Build Question Intent Classifier | Classify exact, semantic, graph, governance, enrichment. | P0 |
| DC-AI-071 | Build Entity Resolver | Resolve mentioned names to canonical entities and jrns. | P0 |
| DC-AI-072 | Build Ambiguity Handling Flow | Ask clarification when multiple assets match. | P1 |
| DC-AI-073 | Build Graph Expansion Service | Expand results through relationships. | P1 |
| DC-AI-074 | Build Context Assembler | Combine chunks, source records, graph paths. | P1 |
| DC-AI-075 | Build Answer Composer | Generate grounded answers from assembled context. | P1 |
| DC-AI-076 | Build Missing Metadata Response Behavior | Return no-answer instead of hallucinating. | P1 |
| DC-AI-077 | Build Lineage Answer Template | Format lineage paths and references. | P1 |

## EPIC 9: Context Plane and Agent Layer

**Description:** Build enterprise context and agent orchestration.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-080 | Build Acronyms Loader | Load and query enterprise acronyms. | P1 |
| DC-AI-081 | Build Business Alias Manager | Map business/technical aliases. | P1 |
| DC-AI-082 | Build Historical Q&A Store | Store validated Q&A with sources. | P1 |
| DC-AI-083 | Build User Bookmarks | User-scoped asset bookmarks. | P2 |
| DC-AI-084 | Build Feedback Capture | Capture helpful/not helpful and source feedback. | P1 |
| DC-AI-085 | Build Supervisor Agent | Route requests to correct retrieval flow. | P1 |
| DC-AI-086 | Build Structured Discovery Agent | Convert vague questions to search plans. | P1 |
| DC-AI-087 | Build Metadata Enrichment Agent | Suggest metadata improvements with evidence. | P2 |
| DC-AI-088 | Build Answer Evaluation Agent | Evaluate groundedness and completeness. | P2 |

## EPIC 10: Security and Access Control

**Description:** Define and implement retrieval-level security controls.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-090 | Define Retrieval Access Control Model | Define role, owner, node, lifecycle, classification rules. | P0 |
| DC-AI-091 | Implement Exact Search Filters | Apply filters to exact search. | P0 |
| DC-AI-092 | Implement Semantic Search Filters | Apply filters to vector search. | P0 |
| DC-AI-093 | Implement Graph Search Filters | Apply filters to graph traversal. | P0 |
| DC-AI-094 | Prevent Restricted Context Assembly | Block restricted records from LLM context. | P0 |
| DC-AI-095 | Implement Audit Logging | Log queries, filters, sources, references. | P1 |
| DC-AI-096 | Build Access-Control Test Suite | Validate negative cases. | P1 |
| DC-AI-097 | Define Sensitive Metadata Handling | Define logging/display rules. | P1 |

## EPIC 11: Evaluation and Quality

**Description:** Build the quality and regression framework.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-100 | Create Golden Question Set | Questions across exact, semantic, graph, governance. | P0 |
| DC-AI-101 | Define Expected Answers | Expected results and source records. | P0 |
| DC-AI-102 | Build Retrieval Quality Evaluation | Measure Top-1, Top-3, Top-5 accuracy. | P1 |
| DC-AI-103 | Build Lineage Correctness Evaluation | Validate graph traversal. | P1 |
| DC-AI-104 | Build Answer Quality Evaluation | Evaluate citations and groundedness. | P1 |
| DC-AI-105 | Build Hallucination Test Cases | Test missing-data behavior. | P1 |
| DC-AI-106 | Build Regression Test Report | Generate recurring reports. | P1 |
| DC-AI-107 | Create Human Review Workflow | Define answer quality review. | P1 |

## EPIC 12: Operations and Monitoring

**Description:** Build dashboards, reindexing, reconciliation, and runbooks.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-110 | Build Indexing Dashboard | Show indexing status by entity/source. | P0 |
| DC-AI-111 | Build Failed Indexing Queue | Show failed records, reasons, retries. | P0 |
| DC-AI-112 | Build Reindex by JRN | Reindex one asset by source jrn. | P1 |
| DC-AI-113 | Build Reindex by Entity Type | Reindex all records for entity type. | P1 |
| DC-AI-114 | Build Vector Count Reconciliation | Compare source records to vector chunks. | P1 |
| DC-AI-115 | Build Graph Count Reconciliation | Compare source records to graph nodes/edges. | P1 |
| DC-AI-116 | Build Search Quality Dashboard | Track quality and failed golden questions. | P1 |
| DC-AI-117 | Create Operational Runbooks | Troubleshooting and support procedures. | P1 |
| DC-AI-118 | Add Cost and Latency Monitoring | Track embedding/search/graph costs and latency. | P2 |

## EPIC 13: UI / User Experience Integration

**Description:** Integrate search, lineage, citations, and feedback into Data Compass UI.

| Story ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-120 | Define Search UI Experience | UX for AI search and result display. | P1 |
| DC-AI-121 | Build Search Result Card | Show entity, lifecycle, source, snippet. | P1 |
| DC-AI-122 | Build Source Citation Display | Show guid/jrn and metadata source link. | P1 |
| DC-AI-123 | Build Lineage Answer Display | Show upstream/downstream paths. | P1 |
| DC-AI-124 | Build Clarification UI | Choose among ambiguous matches. | P2 |
| DC-AI-125 | Build User Feedback UI | Capture answer feedback. | P2 |
| DC-AI-126 | Build Bookmark UI | Allow asset bookmarks. | P2 |
| DC-AI-127 | Build No-Answer State | Explain missing/inaccessible metadata. | P2 |
