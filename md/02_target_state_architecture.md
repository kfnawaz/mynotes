# Target-State Architecture

## 1. Purpose

This page defines the target-state architecture for **Data Compass AI Modernization**.

The target architecture turns Data Compass metadata into an AI-ready, relationship-aware, and governed knowledge platform. It supports exact metadata lookup, semantic discovery, lineage and impact analysis, GraphRAG question answering, metadata enrichment, and operational reindexing.

This page should be used by architecture, engineering, product, security, and operations teams as the reference for how the major components fit together and what each component is responsible for.

## 2. Executive Summary

The target state uses MongoDB as the authoritative store for the normalized Data Compass metadata model. From that source of truth, the platform creates two derived projections:

- **Vector projection** for semantic search over AI-ready metadata chunks.
- **Graph projection** for lineage, dependency traversal, and relationship reasoning.

A Data Compass Search Service provides controlled APIs for exact search, semantic search, graph search, and hybrid search. A GraphRAG Orchestrator combines retrieval results, graph paths, source records, and enterprise context to produce grounded, source-backed answers. The Context Plane provides acronyms, historical Q&A, bookmarks, feedback, and agent orchestration.

The most important architecture rule is:

> MongoDB remains the system of record. Vector indexes and Neo4j are derived, rebuildable projections.


## 3. Target Architecture Goals

| Goal | Description |
| --- | --- |
| AI-ready metadata | Convert structured catalog metadata into meaningful, relationship-aware AI chunks. |
| Source-backed answers | Every answer should be traceable to source metadata using **guid**, **jrn**, entity type, and source collection. |
| Semantic discovery | Allow users to search using business language, acronyms, and natural questions. |
| Exact lookup | Support precise lookup by JRN, GUID, name, schema, table, report ID, owner, or node. |
| Lineage and impact analysis | Use graph traversal to answer upstream/downstream and dependency questions. |
| Governed retrieval | Apply access, lifecycle, and classification filters before context reaches the LLM. |
| Rebuildable projections | Allow vector and graph projections to be rebuilt from authoritative metadata. |
| Deterministic indexing | Re-running index jobs should replace or update known chunks, not create duplicates. |
| Operational transparency | Provide dashboards, reconciliation, failed-record tracking, and reindex controls. |
| Human-controlled enrichment | AI may suggest enrichment, but authoritative metadata updates require review/approval. |

## 4. Target Architecture Principles

1. **MongoDB is authoritative.** MongoDB stores the canonical Data Compass metadata records.
1. **Vector and graph stores are projections.** They are optimized read models, not independent systems of record.
1. **JRN is the relationship identity.** The **jrn** is the stable cross-entity key used for graph nodes, source references, retrieval, and citations.
1. **GUID is the internal source identifier.** The **guid** is retained for traceability, audit, and source-record hydration.
1. **AI chunks are deterministic.** Chunk IDs must be repeatable using source JRN, entity type, chunk type, sequence, and schema version.
1. **Chunk text must be meaningful.** Raw JSON should not be embedded directly. Chunk text should be readable and relationship-aware.
1. **Graph relationships are generated.** Neo4j relationships should be created from canonical metadata, not manually maintained.
1. **Security filters run before LLM context assembly.** Unauthorized metadata must never be passed to the LLM.
1. **Event data is summarized.** High-volume event entities should be summarized before vectorization unless a specific raw-event use case is approved.
1. **AI enrichment requires approval.** AI suggestions should flow through a governed human approval process before updating authoritative metadata.

## 5. Target Logical Architecture

```
+----------------------------------------------------------------------------------+
|                              Data Compass UI / Users                              |
+-------------------------------------------+--------------------------------------+
                                            |
                                            v
+----------------------------------------------------------------------------------+
|                         Context Plane and Agent Layer                             |
|  Acronyms | Historical Q&A | User Bookmarks | Feedback | Supervisor Agent         |
+-------------------------------------------+--------------------------------------+
                                            |
                                            v
+----------------------------------------------------------------------------------+
|                           GraphRAG Orchestration Layer                            |
|  Intent Classification | Entity Resolution | Retrieval Planning | Answer Composer |
+-------------------------------------------+--------------------------------------+
                                            |
                                            v
+----------------------------------------------------------------------------------+
|                         Data Compass Search Service                               |
|        Exact Search | Semantic Search | Graph Search | Hybrid Search              |
+-----------------------------+-----------------------------+------------------------------+
                              |                             |
                              v                             v
+-----------------------------------------+       +----------------------------------+
| MongoDB Atlas Vector / Document Store   |       | Neo4j Graph Projection            |
| AI Chunks | Embeddings | Source Metadata|       | Lineage | Relationships | Paths   |
+-----------------------------+-----------+       +------------------+---------------+
                              ^                                      ^
                              |                                      |
+-----------------------------+--------------------------------------+----------------+
|                        Projection and Indexing Layer                                |
| AI Chunk Builder | Embedding Service | Vector Upsert | Graph Builder | Reindexing   |
+-------------------------------------------+----------------------------------------+
                                            ^
                                            |
+----------------------------------------------------------------------------------+
|                   Canonical / Normalized Data Compass Metadata Model              |
|  Distribution | Dataset | DataModelEntity | Attribute | DataFlow | DataProcess ... |
+-------------------------------------------+----------------------------------------+
                                            ^
                                            |
+----------------------------------------------------------------------------------+
|                         Metadata Ingestion Service                                |
|  Source Reader | Validator | Relationship Resolver | Node Resolver | Change Detect |
+-------------------------------------------+----------------------------------------+
                                            ^
                                            |
+----------------------------------------------------------------------------------+
|                                  Metadata Sources                                 |
|    Snowflake | Databricks | Tableau | Alteryx | Data Compass API | Harvester       |
+----------------------------------------------------------------------------------+
```

## 6. Architecture Layers

### 6.1 Source Layer

The source layer includes systems and crawlers that provide metadata to Data Compass.

| Source | Metadata Examples | Target Handling |
| --- | --- | --- |
| Snowflake | Databases, schemas, tables, columns, query/access history | Normalize into Data Compass entities and lineage/event projections. |
| Databricks | Catalogs, schemas, tables, columns, Unity Catalog lineage/system tables | Normalize into Distribution, Dataset, DataModelEntity, DataModelAttribute, DataFlow, DataProcess. |
| Tableau | Workbooks, dashboards, reports, datasources, fields | Normalize into Report, DataService, Dataset, Distribution relationships where applicable. |
| Alteryx | Workflows, data sources, data transformations | Normalize into DataProcess, DataFlow, input/output distributions. |
| Data Compass API | Existing Data Compass metadata | Use as internal source of truth and enrichment input. |
| Harvester / Crawler | Extracted technical metadata | Validate and normalize before downstream projections. |

### 6.2 Ingestion and Normalization Layer

This layer reads metadata, validates records, resolves relationships, and prepares normalized records.

Key responsibilities:

- Read source records in batch or incremental mode.
- Validate mandatory fields.
- Resolve **guid**, **jrn**, names, qualified names, and source paths.
- Resolve related entities using **jrn** references.
- Resolve Node references using **nodeId** and **nodeType**.
- Track ingestion and indexing status.
- Detect changed, deleted, or deprecated assets.
- Emit normalized records for chunk and graph projection.

### 6.3 Canonical / Normalized Metadata Model Layer

This is the standard Data Compass contract used by all downstream layers.

The model includes major entity families:

- Physical and logical assets: Distribution, Dataset, DataModel, DataModelEntity, DataModelAttribute, DataService, DataProduct, DataDomain.
- Business and semantic entities: BusinessGlossary, BusinessGlossaryTerm, DataConcept, DataElement, qualifier roles and values.
- Lineage and movement: DataFlow, DataProcess, DataContract, DataOffer.
- Event records: consumption, production, process start/complete, data quality, contract breach, compensating action.
- Governance and classification: authority declarations, classification, retention, quality metrics.
- Foundational entities: Node and Report.

MVP will focus on:

- Distribution
- Dataset
- DataModelEntity
- DataModelAttribute
- DataService
- DataFlow
- DataProcess
- Node
- Report

### 6.4 Projection and Indexing Layer

This layer creates optimized read models from canonical metadata.

| Projection | Purpose | Output |
| --- | --- | --- |
| AI chunk projection | Converts structured metadata into readable text chunks | AI-ready chunk documents |
| Vector projection | Embeds AI chunks and stores vectors with metadata filters | Vector index records |
| Graph projection | Converts entities and relationships into Neo4j nodes/edges | Graph database projection |
| Operational projection | Tracks indexing status, failures, retries, and counts | Indexing dashboard data |

### 6.5 Storage and Retrieval Layer

| Store | Responsibility | Source of Truth? | Rebuildable? |
| --- | --- | --- | --- |
| MongoDB Data Compass collections | Authoritative metadata records | Yes | N/A |
| MongoDB AI chunk collection | AI-ready text chunks and metadata | No | Yes |
| MongoDB Atlas Vector Index | Semantic vector search | No | Yes |
| Neo4j Graph Database | Relationship traversal, lineage, impact analysis | No | Yes |
| Redis / Context Store | Session context, cached context, temporary state | No | Yes / Expirable |
| Feedback / Q&A Store | User feedback and validated Q&A | No, unless promoted | Yes |

### 6.6 Search and API Layer

The Search Service provides controlled access to metadata retrieval capabilities.

Core APIs:

- Exact Metadata Search API
- Semantic Vector Search API
- Graph Search API
- Hybrid Search API
- Source Record Hydration API
- Source Citation API
- Metadata Filter API
- Reindex API

### 6.7 GraphRAG Orchestration Layer

The GraphRAG Orchestrator plans and executes retrieval for user questions.

Responsibilities:

- Classify user intent.
- Resolve mentioned entities to canonical entities and JRNs.
- Select retrieval strategy.
- Run exact search, vector search, graph search, or hybrid search.
- Expand relevant graph relationships.
- Hydrate source records from MongoDB.
- Assemble bounded context.
- Generate source-backed answer.
- Ask clarification when entity resolution is ambiguous.
- Refuse to answer or state missing data when evidence is insufficient.

### 6.8 Context Plane and Agent Layer

The Context Plane improves interpretation and answer quality.

Capabilities:

- Acronyms and enterprise terminology.
- Business aliases and synonyms.
- Historical validated Q&A.
- User bookmarks and preferred assets.
- Human feedback.
- Answer self-evaluation.
- Context Plane MCP registry.
- Agent routing and orchestration.

Agents:

| Agent | Responsibility |
| --- | --- |
| Supervisor Agent | Routes user question to the right retrieval path. |
| Structured Discovery Agent | Converts vague search questions into structured search plans. |
| Metadata Enrichment Agent | Suggests missing descriptions, tags, glossary mappings, or relationships. |
| Lineage Analysis Agent | Handles upstream/downstream and impact-analysis questions. |
| Governance Agent | Handles ownership, classification, quality, and policy questions. |
| Answer Evaluation Agent | Checks groundedness, completeness, and source support. |

## 7. Component Responsibility Matrix

| Component | Primary Responsibility | Inputs | Outputs |
| --- | --- | --- | --- |
| Metadata Ingestion Service | Read, validate, normalize, and resolve source metadata | MongoDB/source systems/crawlers | Validated canonical records |
| Relationship Resolver | Resolve **jrn** references across entities | Canonical records | Resolved entity graph references |
| Node Resolver | Resolve **nodeId** and **nodeType** references | Canonical records, Node records | Node-enriched records |
| AI Chunk Builder | Render entity records into meaningful text chunks | Canonical/resolved records | AI chunk documents |
| Embedding Service | Generate vector embeddings for chunks | AI chunk text | Embedding vectors |
| Vector Indexing Service | Upsert embeddings and metadata into vector index | Chunks and embeddings | Searchable vector records |
| Graph Builder | Load graph nodes and relationships into Neo4j | Canonical/resolved records | Neo4j nodes and edges |
| Search Service | Provide exact, semantic, graph, and hybrid search | User query and filters | Ranked source-backed results |
| GraphRAG Orchestrator | Combine retrieval and answer generation | Search results, graph paths, context | Grounded answer |
| Context Plane | Provide enterprise context and feedback loops | Acronyms, bookmarks, Q&A, feedback | Context-enriched query plans |
| Security Layer | Enforce retrieval and context access controls | User identity, entitlements, metadata | Filtered results/context |
| Operations Layer | Monitor indexing, sync, quality, and failures | Job logs, counts, metrics | Dashboards, alerts, runbooks |

## 8. Core Data Contracts

### 8.1 Canonical Entity Minimum Contract

Every MVP entity should expose these common fields to downstream projections where applicable:

| Field | Purpose | Required? |
| --- | --- | --- |
| guid | Internal Data Compass source identity | Yes |
| jrn | Stable relationship and retrieval identity | Yes for linked/searchable entities |
| entity_type | Entity name such as Distribution, Dataset, Report | Yes |
| source_collection | MongoDB collection or logical source | Yes |
| version | Source version for change tracking | Yes if available |
| lifecycleStatus | Search/indexing eligibility | Yes |
| createdTimestamp | Audit and traceability | Yes if available |
| updatedTimestamp | Change detection | Yes if available |
| owners | Access/ranking/ownership | Optional but preferred |
| node | Node reference where applicable | Optional but preferred |
| classification | Governance and security filtering | Optional but preferred |
| relationships | JRN-based references to related entities | Optional but preferred |

### 8.2 AI Chunk Contract

```json
{
  "chunk_id": "<deterministic chunk identifier>",
  "chunk_type": "asset_summary | attribute_summary | lineage_summary | service_summary | classification_summary | process_transformation_summary",
  "text": "<AI-readable metadata text>",
  "source": {
    "guid": "<source guid>",
    "jrn": "<source jrn>",
    "collection": "<source collection>",
    "entity_type": "<entity type>",
    "version": "<source version>",
    "lifecycleStatus": "<lifecycle status>"
  },
  "metadata": {
    "nodeId": "<node id>",
    "nodeType": "<node type>",
    "owners": [],
    "databaseType": "<source platform>",
    "entityPhysicalType": "TABLE | VIEW | MATERIALIZED_VIEW",
    "classification": "PUBLIC | INTERNAL | CONFIDENTIAL | HIGHLY_CONFIDENTIAL",
    "hasPII": false,
    "hasSPI": false,
    "environment": "<env>"
  },
  "relationships": {
    "datasetJrn": "<dataset jrn>",
    "distributionJrns": [],
    "dataServiceJrns": [],
    "upstreamJrns": [],
    "downstreamJrns": []
  },
  "embedding": {
    "model": "<embedding model>",
    "dimensions": 0,
    "embeddedAt": "<timestamp>",
    "schemaVersion": 1
  }
}
```

### 8.3 Graph Node Contract

| Property | Purpose |
| --- | --- |
| jrn | Primary graph identity key |
| guid | Source record traceability |
| entity_type | Node label support and filtering |
| name/displayName | User-facing display |
| lifecycleStatus | Search/traversal filtering |
| source_collection | Hydration path back to MongoDB |
| nodeId/nodeType | Node-based filtering and relationships |
| owners | Ownership and ranking |
| classification | Security filtering |
| updatedTimestamp | Sync and freshness |

### 8.4 Search Request Contract

```json
{
  "query": "<user query>",
  "userContext": {
    "userId": "<user id>",
    "roles": [],
    "entitlements": [],
    "businessUnit": "<business unit>"
  },
  "filters": {
    "entityTypes": [],
    "lifecycleStatuses": [],
    "nodeIds": [],
    "owners": [],
    "classifications": [],
    "sourceSystems": []
  },
  "retrievalMode": "exact | semantic | graph | hybrid | auto",
  "options": {
    "topK": 10,
    "includeGraphExpansion": true,
    "graphDepth": 2,
    "includeSourceRecords": true
  }
}
```

### 8.5 Search Response Contract

```json
{
  "queryId": "<query id>",
  "retrievalModeUsed": "hybrid",
  "results": [
    {
      "title": "<display name>",
      "entityType": "Distribution",
      "sourceGuid": "<guid>",
      "sourceJrn": "<jrn>",
      "lifecycleStatus": "PUBLISHED",
      "score": 0.0,
      "matchedReason": "<why this result matched>",
      "snippet": "<supporting chunk text>",
      "relationships": [],
      "citations": []
    }
  ],
  "warnings": [],
  "clarificationRequired": false,
  "clarificationOptions": []
}
```

## 9. Target Data Flows

### 9.1 Metadata Ingestion Flow

```
Source Systems / MongoDB Collections
    → Metadata Reader
    → Entity Validator
    → Identity Normalizer
    → Relationship Resolver
    → Node Resolver
    → Canonical Metadata Output
    → Indexing Status Update
```

### 9.2 AI Chunk and Vector Indexing Flow

```
Canonical Metadata Record
    → Relationship Enrichment
    → Chunk Template Selection
    → AI-readable Text Rendering
    → Chunk Metadata Assignment
    → Embedding Generation
    → Vector Upsert
    → Indexing Status Update
```

### 9.3 Graph Projection Flow

```
Canonical Metadata Record
    → Relationship Resolution
    → Graph Node Mapping
    → Graph Edge Mapping
    → Neo4j Constraint/Index Check
    → Node/Edge Upsert
    → Graph Sync Status Update
```

### 9.4 Runtime Exact Lookup Flow

```
User Question
    → Intent Classifier
    → Exact Search Service
    → Access Filter
    → Source Record Hydration
    → Answer Composer
    → Source-backed Response
```

### 9.5 Runtime Semantic Discovery Flow

```
User Question
    → Context Plane Enrichment
    → Query Embedding
    → Vector Search with Metadata Filters
    → Result Ranking
    → Source Record Hydration
    → Answer Composer
    → Source-backed Response
```

### 9.6 Runtime Lineage / Impact Flow

```
User Question
    → Intent Classifier
    → Entity Resolver
    → Graph Traversal by JRN
    → Access Filter
    → Source Record Hydration
    → Optional Vector Retrieval for Explanations
    → Lineage Answer Template
    → Source-backed Response
```

### 9.7 Metadata Enrichment Flow

```
User or Scheduled Trigger
    → Candidate Asset Selection
    → Source Metadata + Context Retrieval
    → Enrichment Agent Suggestion
    → Evidence Packaging
    → Human Review
    → Approved Update to Authoritative MongoDB Metadata
    → Reindex / Graph Sync
```

## 10. Retrieval Strategy Matrix

| User Intent | Primary Retrieval | Secondary Retrieval | Expected Behavior |
| --- | --- | --- | --- |
| Find asset by exact name/JRN/GUID | Exact Search | Source hydration | Return exact matching source-backed result. |
| Find assets by business meaning | Semantic Search | Graph expansion | Return semantically relevant assets with source references. |
| Upstream/downstream lineage | Graph Search | Source hydration and vector explanation chunks | Return lineage path and supporting metadata. |
| Impact analysis | Graph Search | Hybrid ranking | Return downstream assets, reports, services, and processes. |
| Governance/classification question | Exact/Semantic Search with filters | Source hydration | Return classification-backed answer. |
| Ambiguous asset name | Entity Resolver | Clarification flow | Ask user to choose among candidates. |
| Missing metadata | Any retrieval mode | N/A | State that data is unavailable; do not invent. |
| Metadata enrichment request | Source hydration + semantic context | Agent suggestion | Provide suggestion with evidence and approval path. |

## 11. Security and Access Control Target State

Security must be enforced at retrieval time and again before context assembly.

### 11.1 Access Filter Inputs

- User identity
- User role
- Entitlements
- Owner relationships
- Node access
- Business unit
- Lifecycle status
- Confidentiality classification
- PII/SPI flags
- Environment/source system

### 11.2 Required Security Controls

| Control | Target Behavior |
| --- | --- |
| Pre-retrieval filters | Apply structured filters before vector and exact search where supported. |
| Post-retrieval validation | Validate retrieved records before context assembly. |
| Graph traversal filtering | Remove unauthorized nodes/edges from graph results. |
| LLM context guard | Prevent restricted metadata from entering prompt/context. |
| Audit logging | Record query, user, filters, source records, and answer references. |
| Sensitive logging rules | Avoid over-logging sensitive values while preserving traceability. |

### 11.3 Default Lifecycle Filter

| lifecycleStatus | Default Retrieval Behavior |
| --- | --- |
| DRAFT | Excluded by default |
| PUBLISHED | Included |
| APPROVED | Included |
| PENDING_APPROVAL | Restricted to authorized users |
| REJECTED | Excluded |
| DEPRECATED | Excluded by default; included only for historical search |

## 12. Graph Target Model

### 12.1 Initial Node Labels

- DataNode
- Dataset
- Distribution
- DataModel
- DataModelEntity
- DataModelAttribute
- DataService
- DataProduct
- DataDomain
- DataFlow
- DataProcess
- Report
- DataElement
- DataConcept
- Owner
- Classification

### 12.2 Initial Relationship Types

| Relationship | From | To | Purpose |
| --- | --- | --- | --- |
| IMPLEMENTS | Distribution | Dataset | Physical asset implements logical dataset. |
| CONTAINS | Distribution/DataModelEntity/Report | Attribute | Parent contains attribute/field. |
| EXPOSES | DataService | Distribution | Service exposes distribution. |
| CONSUMES | DataProduct/DataProcess/Report | Dataset/Distribution | Consumer uses data asset. |
| PRODUCES | DataProduct/DataProcess | Dataset/Distribution | Producer creates data asset. |
| UPSTREAM_OF | Asset | Asset | Lineage direction. |
| DOWNSTREAM_OF | Asset | Asset | Reverse lineage direction. |
| DEPENDS_ON | Report/Process/Product | Asset | Dependency relationship. |
| OWNS | Owner | Asset | Ownership relationship. |
| BELONGS_TO | Asset/Product | Domain | Domain/product hierarchy. |
| MAPS_TO | Attribute | DataElement/DataConcept | Semantic mapping. |
| CLASSIFIED_AS | Asset/Attribute | Classification | Governance classification. |

### 12.3 Graph Query Patterns

- Upstream dependencies by JRN.
- Downstream dependencies by JRN.
- Report impact analysis.
- Data product input/output traversal.
- Data service exposure search.
- Attribute-to-business-term mapping.
- Owner-to-asset lookup.
- Classification-based impact analysis.

## 13. AI Chunk Target Model

### 13.1 Chunk Types by Entity

| Entity | Chunk Types |
| --- | --- |
| Distribution | asset_summary, physical_specification, attribute_summary, service_summary, classification_summary, lineage_summary |
| Dataset | asset_summary, logical_schema_summary, business_mapping_summary, distribution_mapping_summary |
| DataModelEntity | entity_summary, attribute_summary, relationship_summary |
| DataModelAttribute | attribute_summary, classification_summary, semantic_mapping_summary |
| DataService | service_summary, interface_summary, distribution_exposure_summary |
| DataFlow | lineage_summary, provider_consumer_summary, observed_assets_summary |
| DataProcess | process_summary, inbound_outbound_summary, transformation_summary, attribute_lineage_summary |
| Report | report_summary, report_attribute_summary, dependency_summary |
| Node | node_summary, owned_assets_summary, related_services_summary |

### 13.2 Chunk Quality Rules

- Chunk must be readable without requiring raw JSON context.
- Chunk must identify the entity type and asset name.
- Chunk must include source JRN and GUID as metadata.
- Chunk must include parent context where needed.
- Chunk should include relationship context but avoid becoming too large.
- Chunk should not include unauthorized sensitive values.
- Chunk should be versioned and regenerable.

## 14. Non-Functional Requirements

| Category | Requirement | Target / Notes |
| --- | --- | --- |
| Traceability | Every result links to source GUID/JRN | Mandatory |
| Security | Filters before LLM context assembly | Mandatory |
| Rebuildability | Vector and graph projections can be rebuilt | Mandatory |
| Determinism | Reindex does not create duplicate chunks | Mandatory |
| Observability | Index and graph sync status visible | Mandatory |
| Performance | Search latency target to be defined | Open |
| Scalability | Event records should be summarized before vectorization | Required for scale |
| Availability | Search service should be production resilient | Target to be defined |
| Data Quality | Invalid/missing relationships are logged | Mandatory |
| Auditability | Query and retrieval decisions are auditable | Mandatory |
| Explainability | Answers include source references and match reason | Mandatory |
| Maintainability | Component boundaries and contracts documented | Mandatory |

## 15. Deployment and Environment Strategy

### 15.1 Environments

| Environment | Purpose | Notes |
| --- | --- | --- |
| Sandbox / Experimental | Prototype chunking, embeddings, graph schema, and prompts | Can use limited data and synthetic test cases. |
| UAT | Validate entity indexing, graph traversal, access filters, and golden questions | Should mirror production patterns. |
| Production | Serve governed AI metadata search | Requires security, monitoring, runbooks, and reindex controls. |

### 15.2 Promotion Model

```
Sandbox prototype
    → Architecture review
    → UAT implementation
    → Golden question validation
    → Security/access-control validation
    → Production readiness review
    → Production rollout
```

### 15.3 Deployment Units

Possible deployment units:

- Metadata Ingestion Service
- Chunk Builder Worker
- Embedding Worker
- Vector Indexing Worker
- Graph Builder Worker
- Search Service API
- GraphRAG Orchestrator API
- Context Plane Service
- Operations Dashboard

## 16. Operations Target State

### 16.1 Required Operational Capabilities

- View indexing status by entity type.
- View failed ingestion/chunking/embedding/graph records.
- Reindex one asset by JRN.
- Reindex all assets by entity type.
- Rebuild vector index by model version.
- Rebuild graph projection.
- Compare source counts against chunk counts and graph counts.
- Monitor query volume, latency, error rate, and retrieval quality.
- Monitor embedding cost and model usage.
- Maintain runbooks for indexing, search, graph, and access-control issues.

### 16.2 Indexing Status Fields

| Field | Purpose |
| --- | --- |
| source_guid | Source traceability |
| source_jrn | Source relationship identity |
| entity_type | Entity classification |
| source_version | Change detection |
| last_source_updated_at | Freshness |
| chunk_status | Pending, Success, Failed, Skipped |
| embedding_status | Pending, Success, Failed, Skipped |
| vector_sync_status | Pending, Success, Failed |
| graph_sync_status | Pending, Success, Failed |
| last_indexed_at | Operational tracking |
| failure_reason | Troubleshooting |
| retry_count | Operational retry handling |

## 17. Build Sequencing

| Sequence | Capability | Why First / Next |
| --- | --- | --- |
| 1 | Confirm identity and MVP scope | Prevents unstable indexing and graph keys. |
| 2 | Build MongoDB reader and validator | Required for all downstream projections. |
| 3 | Build relationship and node resolver | Required for relationship-aware chunks and graph. |
| 4 | Build AI chunk schema and templates | Required for semantic search quality. |
| 5 | Build embedding and vector indexing | Enables first usable semantic retrieval. |
| 6 | Build exact and semantic search APIs | Provides first end-to-end user-facing capability. |
| 7 | Build Neo4j graph projection | Enables lineage and impact analysis. |
| 8 | Build GraphRAG orchestration | Combines vector, exact, and graph retrieval. |
| 9 | Build Context Plane and agents | Improves interpretation and user experience. |
| 10 | Harden security, evaluation, and operations | Required for production trust. |

## 18. Major Architecture Decisions Required

| ADR | Decision | Status |
| --- | --- | --- |
| ADR-001 | MongoDB remains authoritative source of metadata | Proposed |
| ADR-002 | Vector and graph stores are derived projections | Proposed |
| ADR-003 | JRN is the stable relationship identity key | Proposed |
| ADR-004 | MVP entity scope | Proposed |
| ADR-005 | MongoDB Atlas Vector Search as initial vector store | Proposed |
| ADR-006 | Neo4j as graph projection for lineage and impact analysis | Proposed |
| ADR-007 | AI chunks use deterministic chunk IDs | Proposed |
| ADR-008 | Security filters are applied before LLM context assembly | Proposed |
| ADR-009 | Raw event records are summarized before vectorization | Proposed |
| ADR-010 | AI enrichment suggestions require human approval | Proposed |

## 19. Key Open Questions

| ID | Question | Impact | Needed By |
| --- | --- | --- | --- |
| OQ-001 | Does the crawler mint GUID/JRN or does the catalog DB assign them? | Blocks identity rules | Phase 0 |
| OQ-002 | What is the deterministic JRN format by source platform? | Blocks graph and chunk identity | Phase 0 |
| OQ-003 | Which lifecycle statuses are searchable by default? | Blocks retrieval filters | Phase 0 |
| OQ-004 | What entitlement model controls metadata visibility? | Blocks security design | Phase 1 |
| OQ-005 | Should deprecated assets be retained but filtered or removed from vector index? | Affects reindexing | Phase 1 |
| OQ-006 | Which event entities are summarized in MVP? | Affects cost and scale | Phase 2 |
| OQ-007 | What are search and GraphRAG latency targets? | Affects design and sizing | Phase 2 |
| OQ-008 | What source references must be shown in UI? | Affects API and UX | Phase 2 |
| OQ-009 | What is the approved hosting model for Neo4j? | Platform dependency | Phase 2 |
| OQ-010 | What is the production-approved embedding model? | Security/cost dependency | Phase 2 |

## 20. Success Criteria

### 20.1 Functional Success

- Exact search can find assets by GUID, JRN, name, schema, table, report ID, and owner.
- Semantic search can find assets using business-language questions.
- Graph search can answer upstream and downstream questions by JRN.
- Hybrid search can combine exact, vector, and graph results.
- GraphRAG answers are grounded in source records and retrieved chunks.

### 20.2 Data Quality Success

- Every AI chunk includes source GUID, JRN, entity type, chunk type, and lifecycle status.
- Vector records reconcile with source and chunk counts.
- Graph nodes and edges reconcile with source relationship counts.
- Missing relationships are logged and reportable.

### 20.3 Security Success

- Unauthorized records are filtered before LLM context assembly.
- Deprecated assets are excluded by default.
- Restricted classifications are handled according to policy.
- Query, retrieval, and answer source references are auditable.

### 20.4 User Trust Success

- Answers include source references.
- Ambiguous asset names trigger clarification.
- Missing metadata results in a clear no-answer response.
- Users can provide feedback on answer quality.

### 20.5 Operational Success

- Operators can see indexing status and failures.
- Operators can reindex by JRN and entity type.
- Operators can rebuild graph projection.
- Count reconciliation is available for MongoDB, vector index, and Neo4j.
- Runbooks exist for major failure modes.

## 21. Related Jira Epics

| Epic | Target Architecture Dependency |
| --- | --- |
| EPIC 1: Architecture and Planning | Defines scope, decisions, risks, and target architecture. |
| EPIC 2: Canonical Metadata Foundation | Defines identity, required fields, lifecycle, and relationships. |
| EPIC 3: Metadata Ingestion Service | Provides validated source records. |
| EPIC 4: AI Document and Chunk Builder | Creates AI-ready metadata documents. |
| EPIC 5: Embedding and Vector Indexing | Enables semantic retrieval. |
| EPIC 6: Graph Projection and Neo4j Builder | Enables lineage and relationship traversal. |
| EPIC 7: Data Compass Search Service | Provides controlled retrieval APIs. |
| EPIC 8: GraphRAG Orchestration | Combines exact, semantic, and graph retrieval. |
| EPIC 9: Context Plane and Agent Layer | Improves interpretation, feedback, and enrichment. |
| EPIC 10: Security and Access Control | Enforces governed retrieval. |
| EPIC 11: Evaluation and Quality | Proves correctness and answer quality. |
| EPIC 12: Operations and Monitoring | Provides reliability, observability, and reindex controls. |
| EPIC 13: UI / User Experience Integration | Delivers the user-facing experience. |
