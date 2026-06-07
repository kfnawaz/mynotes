# Canonical / Normalized Metadata Model

## 1. Purpose

This page defines the **Canonical / Normalized Data Compass Metadata Model** for the AI Modernization effort.

The canonical model is the agreed metadata contract used between source crawlers/miners, MongoDB, vector indexing, Neo4j graph projection, Search Service, GraphRAG orchestration, Context Plane, and future metadata-enrichment workflows.

In this context, **canonical** means the single standardized representation of metadata after vendor-specific or source-specific details have been normalized into the Data Compass model.

> **Plain-English Definition**
> The canonical model is the common Data Compass shape that all metadata must follow before it can be chunked, embedded, searched, graphed, enriched, or used by AI.
>
>
## 2. Why This Page Matters
>
> The AI and graph layers will only be trustworthy if they are built from a stable, consistent metadata model.
>
> This page is intended to prevent the following problems:
>
- Vector chunks created directly from inconsistent source payloads
- Neo4j graph edges built from unstable or missing identifiers
- AI answers that cannot be traced back to source metadata
- Duplicate assets caused by inconsistent name or path handling
- Incorrect lineage caused by unresolved relationships
- Security filtering gaps caused by missing owner, node, classification, or lifecycle fields
- Reindexing failures caused by non-deterministic identities
>
## 3. Source Model Summary
>
> Data Compass contains **34 entity types** across the following families:
>
> || Entity Family || Entities || AI / Graph Relevance ||
> | Physical and logical assets | Distribution, Dataset, DataModel, DataModelEntity, DataModelAttribute, DataService, DataProduct, DataDomain | Core asset discovery, schema search, service exposure, product/domain grouping |
> | Business / semantic layer | BusinessGlossary, BusinessGlossaryTerm, DataConcept, DataElement, DataConceptQualifierRole, DataConceptQualifierValue, BusinessContext, BusinessProcess | Business meaning, glossary matching, semantic enrichment |
> | Lineage and movement | DataFlow, DataProcess, DataContract, DataOffer | Lineage, provider/consumer movement, process impact, agreements |
> | Event records | DataConsumptionEvent, DataProductionEvent, DataProcessStartEvent, DataProcessCompleteEvent, DataQualityMeasurementEvent, DataContractBreachEvent, CompensatingActionEvent | Usage, processing, quality, breach intelligence. Summarized before AI indexing in most cases |
> | Governance and classification | DataAuthorityDeclaration, DataNodeRecordClassification, DistributionRecordClassification, DataConceptDefaultRecordClassificationCode, DataQualityMetric | Retention, authority, quality, classification, compliance-aware search |
> | Foundational | Node, Report | Node identity and report dependency discovery |
>
## 4. Design Principle
>
> **Core Design Principle**
> MongoDB remains the authoritative source of Data Compass metadata. Vector indexes and Neo4j are derived, rebuildable projections from the canonical model.
>
>
### What This Means
>
- Canonical MongoDB records are the source of truth.
- AI chunks must link back to canonical records.
- Graph nodes and edges must link back to canonical records.
- Vector records and graph records must be rebuildable.
- No AI-generated enrichment should overwrite canonical metadata without a controlled approval workflow.
- Search and GraphRAG should hydrate source records from MongoDB when preparing source-backed answers.
>
## 5. Canonical Model Responsibilities
>
> The canonical model is responsible for:
>
> || Responsibility || Description ||
> | Identity | Define stable identifiers for each metadata asset |
> | Normalization | Convert source-specific shapes into standard Data Compass entities |
> | Relationship resolution | Represent relationships using stable references, primarily jrn |
> | Lifecycle control | Identify which assets are searchable, deprecated, rejected, or restricted |
> | Ownership | Preserve owners and roles for search, governance, and filtering |
> | Node mapping | Preserve node references used across provider, consumer, broker, and asset relationships |
> | Classification | Preserve PII/SPI/confidentiality/purge/retention indicators |
> | Versioning | Support change detection, reindexing, and stale-record detection |
> | Traceability | Link vector chunks, graph nodes, search results, and AI answers back to source records |
> | Security enablement | Provide metadata required for retrieval-time filtering |
>
## 6. Identity Model
>
### 6.1 GUID
>
> **guid** is the internal Data Compass uniqueness key.
>
> || Usage || Requirement ||
> | Source record identity | Every canonical source record should have a guid where the source entity supports it |
> | MongoDB document traceability | guid should be persisted on canonical records and downstream projections |
> | Reindexing | guid can be used to reindex one source record |
> | Audit | guid should be logged in indexing, search, and answer evidence |
> | Source citation | guid should be included in source reference output where available |
>
### 6.2 JRN
>
> **jrn** is the stable cross-entity relationship key.
>
> || Usage || Requirement ||
> | Relationship resolution | All cross-entity relationships should resolve by jrn where available |
> | AI chunk identity | Every chunk should carry source_jrn |
> | Graph identity | Graph nodes should use jrn as the primary stable key where available |
> | Search result source reference | Results should include source_jrn |
> | Deduplication | Search results should deduplicate by source_jrn |
> | Rebuildability | Repeated crawls should generate the same jrn for the same source asset |
>
### 6.3 GUID vs JRN Rule
>
> || Identifier || Meaning || Primary Use ||
> | guid | Internal Data Compass object identity | Source traceability, audit, reindexing |
> | jrn | Stable resource identity and relationship reference | Relationships, graph nodes, chunk identity, search result deduplication |
>
> **Working Rule**
> Use **jrn** for relationship linking and derived projection identity. Use **guid** for internal record traceability and audit.
>
>
### 6.4 Deterministic Chunk ID Pattern
>
> AI chunks must use deterministic IDs so that reindexing replaces old chunks instead of creating duplicates.
>
> Recommended pattern:
>
```
{source_jrn}:{entity_type}:{chunk_type}:{chunk_sequence}:{chunk_schema_version}
```
>
> Example:
>
```
jrn:databricks:workspace:risk:capital:trade_exposure:Distribution:asset_summary:001:v1
```
>
### 6.5 Deterministic Graph Node Key Pattern
>
> Graph nodes should use a stable canonical key.
>
> Recommended:
>
```
node_key = source_jrn
```
>
> Fallback when jrn is missing:
>
```
node_key = {entity_type}:{source_guid}
```
>
### 6.6 Missing JRN Handling
>
> || Case || Handling ||
> | JRN exists | Use jrn as canonical relationship key |
> | JRN missing but GUID exists | Use guid fallback, mark record with identity_warning |
> | Both JRN and GUID missing | Reject from vector/graph projection and route to validation error queue |
> | JRN changes unexpectedly | Flag as identity drift and require review |
> | Duplicate JRN detected | Block projection until duplicate is resolved |
>
## 7. Canonical Entity Scope
>
### 7.1 MVP Entity Scope
>
> MVP should focus on the entities needed for asset discovery, column search, service exposure, and basic lineage.
>
> || Entity || MVP? || Rationale ||
> | Distribution | Yes | Primary physical data asset. Critical for table/view discovery and lineage |
> | Dataset | Yes | Logical data asset. Needed to connect physical assets to business meaning |
> | DataModelEntity | Yes | Table/view/entity-level schema representation |
> | DataModelAttribute | Yes | Column/attribute search and classification discovery |
> | DataService | Yes | Service/API/database exposure of data assets |
> | DataFlow | Yes | Observed lineage and provider-consumer movement |
> | DataProcess | Yes | Process and transformation lineage |
> | Node | Yes | Foundational relationship entity referenced across the model |
> | Report | Yes | Downstream reporting dependency discovery |
> | DataProduct | Phase 2 | Product grouping and input/output asset relationships |
> | DataDomain | Phase 2 | Domain hierarchy and enterprise data product organization |
> | BusinessGlossary / Terms | Phase 2 | Business-language search and glossary enrichment |
> | DataConcept / DataElement | Phase 2 | Semantic mapping and attribute meaning |
> | DataContract / DataOffer | Phase 2/3 | Agreements, commitments, quality and usage expectations |
> | Governance / Classification entities | Phase 2/3 | Retention, authority, classification, compliance-aware search |
> | Event entities | Phase 3 | High-volume runtime intelligence. Should generally be summarized before indexing |
>
### 7.2 MVP Entity Rationale
>
> The MVP deliberately starts with entities that support the most common high-value questions:
>
- What does this asset represent?
- Which tables/views exist for a subject area?
- Which columns contain a specific business concept or classification?
- Who owns this asset?
- What services expose this data?
- What is upstream/downstream of this asset?
- Which process creates or consumes this data?
- Which reports depend on this asset?
>
## 8. Shared Managed Item Base
>
> Most primary entities should carry the standard managed item header.
>
> || Field || Required for Projection? || Purpose ||
> | guid | Yes, if available | Internal identity and traceability |
> | version | Yes | Change tracking and reindexing |
> | lifecycleStatus | Yes | Search inclusion/exclusion and governance |
> | createdTimestamp | Yes | Audit and source freshness |
> | createdBy | Yes, if available | Source creator traceability |
> | updatedTimestamp | Yes, if available | Change detection |
> | updatedBy | No | Update audit |
> | certifiedTimestamp | No | Ranking and trust signal |
> | certifiedBy | No | Governance traceability |
> | jrn | Yes for relationship-bearing entities | Stable relationship and projection identity |
> | owners[] | No, but strongly preferred | Access control, ranking, stewardship |
> | owners[].sid | Required when owners section exists | Owner identity |
> | owners[].role | Required when role exists | Governance and access filtering |
>
## 9. Lifecycle Handling
>
> Lifecycle status determines search visibility and projection behavior.
>
> || lifecycleStatus || Vector Index Behavior || Graph Behavior || Search Behavior || Notes ||
> | DRAFT | Excluded by default | Optional, excluded by default | Not searchable unless explicitly enabled | Useful for admin preview only |
> | PUBLISHED | Indexed | Projected | Searchable | Default searchable state |
> | APPROVED | Indexed | Projected | Searchable | High-trust state |
> | PENDING_APPROVAL | Restricted indexing or filtered | Optional | Restricted to authorized users | Useful for review workflows |
> | REJECTED | Excluded | Excluded | Not searchable | Should not appear in AI answers |
> | DEPRECATED | Deactivated or retained with filter | Optional/historical | Excluded by default | Available only for historical search if enabled |
>
### Lifecycle Rules
>
- Default search should include only PUBLISHED and APPROVED assets.
- DEPRECATED assets should not appear unless historical search is explicitly requested.
- REJECTED assets should not be indexed for normal AI search.
- DRAFT assets should not be available to general users.
- Lifecycle status must be stored in chunk metadata and graph node properties.
>
## 10. Node Reference Pattern
>
> Node is a foundational entity. Many entities reference nodes through a pair:
>
- node.nodeId
- node.nodeType
>
> Common node reference locations include:
>
- node
- dataProvider.node
- dataConsumer.node
- eventBrokerNode
- performingNodes[]
>
### Node Types
>
> || nodeType || Meaning ||
> | EXTERNAL_SOURCE | External system or data source |
> | INTELLIGENT_SOLUTION | Intelligent/AI system |
> | SEAL | SEAL node / system context |
> | USER_TOOL | User-facing tool |
> | OTHER | Other node type |
>
### Node Resolution Rules
>
> || Rule || Description ||
> | NR-001 | Every node pair must preserve nodeId and nodeType |
> | NR-002 | If a Node entity exists, attach Node metadata to projection records |
> | NR-003 | If Node cannot be resolved, preserve raw nodeId/nodeType and log warning |
> | NR-004 | Node references must be available for security filtering and graph traversal |
> | NR-005 | Node information should be included in AI chunks where it helps interpret ownership/provider/consumer context |
>
## 11. Ownership and Role Handling
>
> Owners are important for stewardship, ranking, filtering, and accountability.
>
### Owner Fields
>
> || Field || Meaning ||
> | owners[].sid | Worker or service identity |
> | owners[].role | Governance/stewardship role |
>
### Owner Role Examples
>
> Common roles include:
>
- ADMIN
- GUEST
- CT_IA
- CT_GOV
- CDO
- CDO_DELEGATE
- RO
- RO_DELEGATE
- BPO
- BPO_DELEGATE
- CTO
- TOWER_IA
- DDO
- DPM
- DSO
- DO
- DE
>
### Owner Rules
>
> || Rule || Description ||
> | OR-001 | Preserve owner SID values exactly as source metadata provides them |
> | OR-002 | Normalize owner role casing to approved enum values |
> | OR-003 | Include owners in vector metadata when available |
> | OR-004 | Include ownership edges in graph projection when useful |
> | OR-005 | Use ownership as a ranking and access-control signal, not as the only security mechanism |
>
## 12. Classification and Data Protection Handling
>
> The canonical model must preserve classification metadata because AI search must not retrieve or expose restricted content incorrectly.
>
### Common Classification Fields
>
> || Field || Type || Purpose ||
> | isPersonalInformation | Boolean | Indicates personal information |
> | isSensitivePersonalInformation | Boolean | Indicates sensitive personal information |
> | isClientConfidentialInformation | Boolean | Indicates client confidential information |
> | isMaterialNonPublicInformation | Boolean | Indicates MNPI |
> | isDataSubjectAccessRequestIndicator | Boolean | Indicates DSAR relevance |
> | isPurgeIndicator | Boolean | Indicates purge requirement |
> | informationConfidentialityClassification | Enum | PUBLIC, INTERNAL, CONFIDENTIAL, HIGHLY_CONFIDENTIAL |
>
### Classification Rules
>
> || Rule || Description ||
> | CL-001 | Classification fields must be preserved on chunks where present |
> | CL-002 | Classification fields must be available as vector metadata filters |
> | CL-003 | Classification fields should be graph node properties where relevant |
> | CL-004 | Attribute-level classification must roll up to parent asset summaries when useful |
> | CL-005 | HIGHLY_CONFIDENTIAL and PII/SPI metadata must be filtered based on user access policy before LLM context assembly |
> | CL-006 | Classification flags should be included in answer evidence where appropriate |
>
## 13. Canonical Relationship Map
>
> Relationships are primarily expressed through jrn reference fields.
>
### Core MVP Relationships
>
> || Source Entity || Relationship || Target Entity || Projection Need ||
> | Dataset | implemented by / represented by | Distribution | Vector context and graph edge |
> | Distribution | belongs to / represents | Dataset | Asset search and graph traversal |
> | Distribution | contains | DataModelAttribute / physical attributes | Attribute search and classification rollup |
> | DataModelEntity | contains | DataModelAttribute | Schema graph and column search |
> | DataService | exposes / accesses | Distribution | Service exposure search |
> | DataFlow | provider to consumer | Node / DataService / Asset | Observed lineage |
> | DataProcess | consumes | Distribution | Upstream/process lineage |
> | DataProcess | produces | Asset / Distribution | Downstream/process lineage |
> | Report | depends on | Dataset / Distribution / Attributes | Report impact analysis |
> | Node | owns / participates in | Assets, flows, processes | Provider/consumer/security context |
>
### Phase 2/3 Relationships
>
> || Source Entity || Relationship || Target Entity || Projection Need ||
> | DataProduct | consumes / produces | Dataset / DataService / Distribution | Product-level discovery |
> | DataDomain | parent / child | DataDomain / DataProduct | Domain hierarchy |
> | DataModelAttribute | maps to | DataElement | Business meaning |
> | DataElement | belongs to | DataConcept | Semantic context |
> | DataContract | agreement between | Provider / Consumer / DataAsset | Governance and usage expectations |
> | DataOffer | defines commitments for | DataAsset | Quality/timeliness/schema commitments |
> | DataQualityMetric | assesses / references | Artifacts | Quality-aware search |
> | Classification entities | classify | Node / Distribution / DataConcept | Retention and compliance |
>
## 14. Canonical to AI Chunk Projection
>
> The canonical model feeds the AI chunk builder.
>
### Chunk Projection Rules
>
> || Rule || Description ||
> | AIC-001 | Every chunk must include source_guid when available |
> | AIC-002 | Every chunk must include source_jrn when available |
> | AIC-003 | Every chunk must include entity_type and chunk_type |
> | AIC-004 | Every chunk must include lifecycleStatus |
> | AIC-005 | Every chunk must include classification metadata when present |
> | AIC-006 | Every chunk must include relationship context where it improves retrieval |
> | AIC-007 | Chunk text must be human-readable and meaningful without raw JSON |
> | AIC-008 | Chunk IDs must be deterministic |
>
### Recommended Chunk Types by Entity
>
> || Entity || Recommended Chunk Types ||
> | Distribution | asset_summary, physical_specification, attribute_summary, classification_summary, lineage_summary, service_summary |
> | Dataset | dataset_summary, logical_schema_summary, business_context_summary, related_distribution_summary |
> | DataModelEntity | entity_summary, entity_attribute_summary |
> | DataModelAttribute | attribute_summary, classification_summary, business_mapping_summary |
> | DataService | service_summary, interface_summary, exposed_distribution_summary |
> | DataFlow | lineage_summary, provider_consumer_summary |
> | DataProcess | process_summary, transformation_summary, inbound_outbound_summary |
> | Report | report_summary, report_attribute_summary, dependency_summary |
> | Node | node_summary, related_asset_summary |
>
### Standard AI Chunk Contract
>
```json
{
  "chunk_id": "",
  "chunk_type": "asset_summary",
  "text": "Human-readable AI-ready chunk text",
  "source": {
    "guid": "",
    "jrn": "",
    "collection": "Distribution",
    "entity_type": "Distribution",
    "version": 1,
    "lifecycleStatus": "PUBLISHED"
  },
  "metadata": {
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
    "isDeprecated": false
  },
  "relationships": {
    "datasetJrn": "",
    "distributionJrns": [],
    "dataServiceJrns": [],
    "upstreamJrns": [],
    "downstreamJrns": [],
    "reportJrns": [],
    "dataProductJrns": []
  },
  "embeddingInfo": {
    "model": "",
    "dimensions": 0,
    "chunkSchemaVersion": "v1",
    "embeddedAt": ""
  }
}
```
>
## 15. Canonical to Graph Projection
>
> The canonical model also feeds Neo4j graph projection.
>
### Graph Projection Rules
>
> || Rule || Description ||
> | GP-001 | Graph nodes must use jrn as the primary node key where available |
> | GP-002 | Graph nodes must include source_guid where available |
> | GP-003 | Graph nodes must include entity_type and lifecycleStatus |
> | GP-004 | Graph edges must be generated from canonical relationships, not manually curated |
> | GP-005 | Graph projection must be rebuildable from source metadata |
> | GP-006 | Graph loaders must be idempotent |
> | GP-007 | Relationship failures must be logged and reconcilable |
> | GP-008 | Graph traversal queries must return source references |
>
### Recommended Graph Labels
>
> || Label || Source Entity || Key Properties ||
> | Node | Node | jrn/guid, nodeId, nodeType, lifecycleStatus |
> | Dataset | Dataset | jrn, guid, name, lifecycleStatus |
> | Distribution | Distribution | jrn, guid, store, schema, physicalType, lifecycleStatus |
> | DataModelEntity | DataModelEntity | jrn, guid, entityPhysicalType, schema, store |
> | DataModelAttribute | DataModelAttribute | jrn, guid, name, dataType, classification flags |
> | DataService | DataService | jrn, guid, interfaceType, type, authenticationType |
> | DataFlow | DataFlow | jrn, guid, precision, lastObservedTimestamp |
> | DataProcess | DataProcess | jrn, guid, executionFrequency, executionMethod |
> | Report | Report | jrn, guid, reportIdentificationNumber, productionFrequency |
> | Owner | Derived from owners[] | sid, role |
>
### Recommended Graph Relationships
>
> || Relationship || From || To || Purpose ||
> | IMPLEMENTS | Distribution | Dataset | Physical asset implements logical dataset |
> | CONTAINS | Distribution/DataModelEntity | DataModelAttribute | Attribute search and lineage |
> | EXPOSES | DataService | Distribution | Service exposure |
> | CONSUMES | DataProcess/DataProduct | Distribution/Dataset | Input dependency |
> | PRODUCES | DataProcess/DataProduct | Distribution/Dataset | Output dependency |
> | DEPENDS_ON | Report | Distribution/Dataset | Report impact analysis |
> | UPSTREAM_OF | Asset | Asset | Lineage traversal |
> | DOWNSTREAM_OF | Asset | Asset | Lineage traversal |
> | OWNS | Owner | Asset | Ownership/stewardship |
> | PARTICIPATES_IN | Node | Flow/Process | Provider/consumer context |
> | MAPS_TO | DataModelAttribute | DataElement | Business semantic mapping |
> | CLASSIFIED_AS | Asset/Attribute | Classification | Governance context |
>
## 16. Canonical Validation Rules
>
### Validation Levels
>
> || Level || Description || Action on Failure ||
> | P0 Critical | Required for identity, traceability, or projection | Reject from vector/graph projection |
> | P1 High | Required for good search or graph quality | Project with warning if safe |
> | P2 Medium | Useful enrichment metadata | Project and log quality warning |
>
### Minimum Validation Rules
>
> || Rule ID || Rule || Level ||
> | VAL-001 | Entity type must be recognized | P0 |
> | VAL-002 | Record must have guid or jrn | P0 |
> | VAL-003 | Relationship-bearing records must have jrn or a documented fallback | P0 |
> | VAL-004 | lifecycleStatus must be valid enum value | P0 |
> | VAL-005 | version must be present where entity supports version | P1 |
> | VAL-006 | updatedTimestamp should be present for change detection | P1 |
> | VAL-007 | node reference should include nodeId and nodeType when present | P1 |
> | VAL-008 | owners[].sid must exist when owners[] is populated | P1 |
> | VAL-009 | classification enum values must be valid | P1 |
> | VAL-010 | referenced jrn values should resolve to target entities | P1 |
> | VAL-011 | duplicate jrn values must be flagged | P0 |
> | VAL-012 | deprecated/rejected assets must not be searchable by default | P0 |
>
## 17. Entity-Specific Canonical Requirements for MVP
>
### Distribution
>
> Purpose: Physical instantiation of a Dataset and primary crawler target.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include physicalSpecification | Store type/name/schema/type/resource URI |
> | Should include dataset reference | Used to connect logical and physical asset |
> | Should include dataAttributes[] | Used for attribute chunks and graph CONTAINS edges |
> | Should include owners[] where available | Stewardship and access/ranking |
> | Should include classification block where available | Security and governance |
> | Should include node reference where available | Provider/system context |
>
> Recommended chunks:
>
- asset_summary
- physical_specification
- attribute_summary
- classification_summary
- lineage_summary
- service_summary
>
> Recommended graph nodes/edges:
>
- (:Distribution)
- (:Distribution)-[:IMPLEMENTS]->(:Dataset)
- (:Distribution)-[:CONTAINS]->(:DataModelAttribute)
- (:DataService)-[:EXPOSES]->(:Distribution)
>
### Dataset
>
> Purpose: Logical data asset.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include boundarySets[] | Business concept boundaries |
> | Should include conceptual/logical model references | Semantic and schema context |
> | Should link to Distributions | Physical implementations |
> | Should include classification where applicable | Governance-aware discovery |
>
> Recommended chunks:
>
- dataset_summary
- logical_schema_summary
- business_context_summary
- related_distribution_summary
>
### DataModelEntity
>
> Purpose: Entity/table/view-level model object.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include entityPhysicalType | TABLE/VIEW/MATERIALIZED_VIEW |
> | Should include entityStoreName and entitySchemaName | Search and filtering |
> | Should include dataModelAttributes[] | Attribute graph and chunks |
> | Should include dataConcepts[] where available | Semantic mapping |
>
> Recommended chunks:
>
- entity_summary
- entity_attribute_summary
>
### DataModelAttribute
>
> Purpose: Attribute/column-level metadata.
>
> || Requirement || Notes ||
> | Must include guid or jrn, or parent-derived identity | Required for attribute projection |
> | Must include name | Required for column search |
> | Should include dataType | Attribute understanding |
> | Should include position | Schema ordering |
> | Should include key flags | Primary/foreign/index/unique/nullability |
> | Should include dataProtection block | Security filtering |
> | Should include dataElements[] where available | Business meaning |
>
> Recommended chunks:
>
- attribute_summary
- classification_summary
- business_mapping_summary
>
### DataService
>
> Purpose: Service/API/database/storage access to data.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include interfaceType | API/DATABASE/STORAGE/GENERIC |
> | Should include type | PROVIDER/CONSUMER |
> | Should include authenticationType | Security context |
> | Should include dataDistributions[] | Service-to-asset relationship |
> | Should include interface-specific details | API/database/kafka/storage details |
>
> Recommended chunks:
>
- service_summary
- interface_summary
- exposed_distribution_summary
>
### DataFlow
>
> Purpose: Observed lineage from provider to consumer.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include dataProvider | Provider node/service context |
> | Should include dataConsumer | Consumer node/service context |
> | Should include dataAssets[] | Assets transferred |
> | Should include precision | CONCEPTUAL/FUNCTIONAL/TECHNICAL |
> | Should include lastObservedTimestamp | Freshness and ranking |
>
> Recommended chunks:
>
- lineage_summary
- provider_consumer_summary
>
### DataProcess
>
> Purpose: Processing/transformation lineage.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include processGroup | Process grouping |
> | Should include executionFrequency and executionMethod | Operational context |
> | Should include inboundDataDistributions[] | Upstream dependency |
> | Should include outboundDataAsset | Output dependency |
> | Should include transformations[] | Attribute-level lineage and business logic |
>
> Recommended chunks:
>
- process_summary
- inbound_outbound_summary
- transformation_summary
>
### Node
>
> Purpose: Foundational referenced node behind provider/consumer/broker/system relationships.
>
> || Requirement || Notes ||
> | Must include guid or nodeId/nodeType | Required for resolution |
> | Must include type/nodeType | Used for filtering and graph context |
> | Should include jrn if available | Stable graph identity |
>
> Recommended chunks:
>
- node_summary
- related_asset_summary
>
### Report
>
> Purpose: Reporting artifact and downstream dependency target.
>
> || Requirement || Notes ||
> | Must include guid or jrn | Required for projection |
> | Should include reportIdentificationNumber | Exact lookup |
> | Should include productionFrequency | Operational context |
> | Should include node reference | Owning/reporting node |
> | Should include reportSpecification.reportAttributes[] | Report field search |
> | Should include dependencies where available | Impact analysis |
>
> Recommended chunks:
>
- report_summary
- report_attribute_summary
- dependency_summary
>
## 18. Change Detection and Versioning
>
> The canonical model must support deterministic incremental indexing.
>
### Change Detection Inputs
>
- version
- updatedTimestamp
- updatedBy
- lifecycleStatus
- hash of projection-relevant fields
- relationship change hash
>
### Projection Versioning
>
> || Version Type || Purpose ||
> | Source version | Tracks source metadata changes |
> | Chunk schema version | Tracks AI chunk template changes |
> | Embedding model version | Tracks embedding regeneration need |
> | Graph schema version | Tracks graph rebuild need |
> | Relationship resolver version | Tracks relationship projection changes |
>
### Reindex Triggers
>
> || Trigger || Action ||
> | Source record updated | Rebuild affected chunks and graph node/edges |
> | Related record updated | Rebuild relationship-aware chunks |
> | Classification changed | Rebuild asset and attribute chunks; update filters |
> | Lifecycle changed | Update vector/graph/search visibility |
> | Chunk template changed | Rebuild affected chunk types |
> | Embedding model changed | Re-embed affected chunks |
> | Graph schema changed | Rebuild graph projection |
>
## 19. Relationship-Aware Reindexing
>
> A change in one entity may require reindexing related entities.
>
> || Changed Entity || Also Reindex / Reproject || Why ||
> | DataModelAttribute | Parent DataModelEntity, Distribution, Dataset | Attribute summary and classification rollup |
> | Distribution | Dataset, DataService, DataFlow, DataProcess, Report | Asset context and lineage changes |
> | Dataset | Distribution, DataProduct, Report | Logical asset meaning changes |
> | DataService | Related Distribution | Exposure context changes |
> | DataFlow | Provider/consumer assets | Lineage context changes |
> | DataProcess | Inbound/outbound assets | Process lineage changes |
> | Node | Related assets/flows/processes | Node context and access filters |
> | Classification record | Classified asset/attribute | Governance and search filters |
>
## 20. Search and Retrieval Implications
>
> The canonical model enables three search modes.
>
> || Search Mode || Depends On Canonical Fields || Example ||
> | Exact search | guid, jrn, name, schema, report ID, owner | Find table by JRN |
> | Semantic search | AI chunk text + metadata | Find datasets related to liquidity risk |
> | Graph search | jrn relationships and graph edges | What is downstream of this distribution? |
>
### Required Metadata Filters
>
> The following metadata should be available to the Search Service:
>
- entity_type
- source_collection
- source_guid
- source_jrn
- lifecycleStatus
- owners
- ownerRoles
- nodeId
- nodeType
- databaseType
- entityStoreName
- entitySchemaName
- entityPhysicalType
- confidentiality classification
- PII/SPI/MNPI indicators
- updatedTimestamp
- certifiedTimestamp where available
>
## 21. Security Implications
>
> The canonical model must provide enough metadata for security filtering before LLM context assembly.
>
### Security Rules
>
> || Rule || Description ||
> | SEC-MODEL-001 | lifecycleStatus must be available for filtering |
> | SEC-MODEL-002 | classification flags must be available for filtering |
> | SEC-MODEL-003 | nodeId/nodeType must be available where relevant |
> | SEC-MODEL-004 | owner SID/role must be available where relevant |
> | SEC-MODEL-005 | restricted records must not be passed to the LLM context layer |
> | SEC-MODEL-006 | audit logs must include source GUID/JRN for retrieved records |
>
## 22. Data Quality Scorecard
>
> The platform should track canonical model quality by entity type.
>
> || Metric || Description || Target for MVP ||
> | Records with guid | % of records with guid | 95%+ |
> | Records with jrn | % of relationship-bearing records with jrn | 95%+ |
> | Valid lifecycleStatus | % of records with valid lifecycle status | 99%+ |
> | Relationship resolution rate | % of jrn references resolved | 90%+ initially, improve over time |
> | Owner coverage | % of records with owner information | Baseline then improve |
> | Node resolution rate | % of node references resolved | 90%+ initially |
> | Classification coverage | % of attribute/asset records with classification metadata | Baseline then improve |
> | Projection failure rate | % of records failing vector/graph projection | <5% MVP |
> | Duplicate JRN count | Number of duplicate JRN collisions | 0 target |
>
## 23. Example Canonical Projection
>
### Source-Like Distribution Input
>
```json
{
  "guid": "guid-123",
  "jrn": "jrn:databricks:workspace:risk:capital:trade_exposure",
  "version": 4,
  "lifecycleStatus": "PUBLISHED",
  "owners": [{"sid": "S123456", "role": "CDO"}],
  "physicalSpecification": {
    "entityStoreType": "DATABRICKS",
    "entityStoreName": "risk",
    "entitySchemaName": "capital",
    "entityPhysicalType": "TABLE",
    "entityResourceURI": "databricks://risk/capital/trade_exposure"
  },
  "dataset": {
    "jrn": "jrn:dataset:trade_exposure"
  }
}
```
>
### AI Chunk Projection
>
```json
{
  "chunk_id": "jrn:databricks:workspace:risk:capital:trade_exposure:Distribution:asset_summary:001:v1",
  "chunk_type": "asset_summary",
  "text": "Distribution trade_exposure is a physical Databricks TABLE in store risk and schema capital. It implements Dataset trade_exposure and is owned by S123456 with role CDO. Lifecycle status is PUBLISHED.",
  "source": {
    "guid": "guid-123",
    "jrn": "jrn:databricks:workspace:risk:capital:trade_exposure",
    "collection": "Distribution",
    "entity_type": "Distribution",
    "version": 4,
    "lifecycleStatus": "PUBLISHED"
  },
  "metadata": {
    "owners": ["S123456"],
    "ownerRoles": ["CDO"],
    "databaseType": "DATABRICKS",
    "entityStoreName": "risk",
    "entitySchemaName": "capital",
    "entityPhysicalType": "TABLE"
  },
  "relationships": {
    "datasetJrn": "jrn:dataset:trade_exposure"
  }
}
```
>
### Graph Projection
>
```
MERGE (d:Distribution {jrn: 'jrn:databricks:workspace:risk:capital:trade_exposure'})
SET d.guid = 'guid-123',
    d.lifecycleStatus = 'PUBLISHED',
    d.entityStoreType = 'DATABRICKS',
    d.entityStoreName = 'risk',
    d.entitySchemaName = 'capital',
    d.entityPhysicalType = 'TABLE'
MERGE (ds:Dataset {jrn: 'jrn:dataset:trade_exposure'})
MERGE (d)-[:IMPLEMENTS]->(ds)
```
>
## 24. Build Requirements
>
> || ID || Requirement || Priority || Related Jira ||
> | CAN-001 | Confirm MVP entity scope | P0 | DC-AI-010 |
> | CAN-002 | Define GUID and JRN identity rules | P0 | DC-AI-011 |
> | CAN-003 | Define deterministic chunk ID format | P0 | DC-AI-012 |
> | CAN-004 | Define required fields for MVP entities | P0 | DC-AI-013 |
> | CAN-005 | Define relationship resolution rules | P0 | DC-AI-014 |
> | CAN-006 | Define lifecycle handling | P0 | DC-AI-015 |
> | CAN-007 | Define classification metadata handling | P0 | DC-AI-016 |
> | CAN-008 | Define validation rules and failure handling | P0 | DC-AI-021 |
> | CAN-009 | Define node resolution rules | P0 | DC-AI-023 |
> | CAN-010 | Define owner resolution rules | P1 | DC-AI-024 |
> | CAN-011 | Define projection contracts for AI chunks | P0 | DC-AI-030 |
> | CAN-012 | Define projection contracts for Neo4j graph | P1 | DC-AI-050 |
> | CAN-013 | Define change detection and reindex triggers | P1 | DC-AI-026 |
>
## 25. Acceptance Criteria
>
> || Category || Acceptance Criteria ||
> | Scope | MVP entity scope is approved and documented |
> | Identity | GUID/JRN rules are documented and accepted |
> | Relationships | Relationship resolution rules are documented for MVP entities |
> | Lifecycle | Search and projection behavior by lifecycle status is documented |
> | Classification | Classification fields and filter behavior are documented |
> | AI Projection | Standard AI chunk contract is documented |
> | Graph Projection | Graph node/edge projection rules are documented |
> | Validation | Critical validation rules and failure handling are documented |
> | Reindexing | Change detection and relationship-aware reindexing rules are documented |
> | Security | Required fields for retrieval-time access control are documented |
>
## 26. Pitfalls and Mitigations
>
> || Pitfall || Impact || Mitigation ||
> | Unstable JRN | Broken graph edges, duplicate chunks, unreliable citations | Define deterministic JRN rules before build |
> | Missing JRN | Relationship resolution failure | Use guid fallback only where safe and log exception |
> | Duplicate JRN | Incorrect merge of distinct assets | Block projection and require remediation |
> | Embedding raw JSON | Poor semantic quality | Use AI-readable text chunk templates |
> | Missing lifecycle metadata | Deprecated/rejected assets may appear in search | Require lifecycleStatus in projections |
> | Missing classification fields | Security and compliance risk | Preserve classification metadata and enforce filters |
> | Incomplete relationship resolver | Weak lineage answers | Build resolver before GraphRAG |
> | Graph becomes source of truth | Data inconsistency | Treat graph as rebuildable projection |
> | AI enrichment overwrites metadata | Governance risk | Require human approval for updates |
> | No reindex strategy | Stale AI and graph results | Track versions, hashes, and projection status |
>
## 27. Open Questions
>
> || ID || Question || Impact || Owner || Status ||
> | OQ-001 | Does crawler mint guid and jrn, or does catalog DB assign them? | Blocks final identity design | TBD | Open |
> | OQ-002 | What is the deterministic JRN format for Databricks, Snowflake, Tableau, and other sources? | Blocks source-specific identity implementation | TBD | Open |
> | OQ-003 | Which lifecycle statuses should be searchable by default in production? | Blocks retrieval filters | TBD | Open |
> | OQ-004 | How should deprecated assets be represented: retained and filtered, or removed from vector/graph projections? | Affects reindexing and historical search | TBD | Open |
> | OQ-005 | What user entitlement source controls access to owners, nodes, and classifications? | Blocks security integration | TBD | Open |
> | OQ-006 | Should attribute-level JRN be minted if source does not provide it? | Affects column search and graph identity | TBD | Open |
> | OQ-007 | Should unresolved relationships still be indexed as partial context? | Affects recall vs trust | TBD | Open |
> | OQ-008 | What is the minimum metadata required for a record to be AI-searchable? | Affects validation thresholds | TBD | Open |
> | OQ-009 | What is the required audit detail for each AI answer source reference? | Affects citation and logging design | TBD | Open |
> | OQ-010 | Should DataProduct tactical distribution fields remain in target design? | Affects product relationship projection | TBD | Open |
>
## 28. Related ADRs
>
> || ADR || Decision || Status ||
> | ADR-001 | MongoDB remains authoritative source of metadata | Proposed |
> | ADR-002 | Vector and graph stores are derived projections | Proposed |
> | ADR-003 | JRN is the stable relationship identity key | Proposed |
> | ADR-004 | MVP entity scope | Proposed |
> | ADR-007 | AI chunks use deterministic chunk IDs | Proposed |
> | ADR-008 | Security filters are applied before LLM context assembly | Proposed |
> | ADR-010 | AI enrichment suggestions require human approval | Proposed |
>
## 29. Related Jira Epics and Stories
>
### Related Epics
>
- EPIC 2: Canonical Metadata Foundation
- EPIC 3: Metadata Ingestion Service
- EPIC 4: AI Document and Chunk Builder
- EPIC 6: Graph Projection and Neo4j Builder
- EPIC 10: Security and Access Control
- EPIC 12: Operations and Monitoring
>
### Related Stories
>
> || Jira ID || Story ||
> | DC-AI-010 | Confirm MVP Entity Scope |
> | DC-AI-011 | Define GUID and JRN Identity Rules |
> | DC-AI-012 | Define Deterministic Chunk ID Format |
> | DC-AI-013 | Define Canonical Field Requirements |
> | DC-AI-014 | Define Relationship Resolution Rules |
> | DC-AI-015 | Define Lifecycle Status Handling |
> | DC-AI-016 | Define Classification Metadata Handling |
> | DC-AI-021 | Build Entity Validator |
> | DC-AI-022 | Build JRN Relationship Resolver |
> | DC-AI-023 | Build Node Resolver |
> | DC-AI-024 | Build Owner Resolver |
> | DC-AI-030 | Define AI Chunk Schema |
> | DC-AI-050 | Define Neo4j Graph Schema |
>
## 30. First Decisions to Lock
>
> The following decisions should be finalized before vector or graph implementation begins:
>
1. Confirm whether crawler or catalog DB owns guid and jrn creation.
1. Confirm deterministic JRN format by platform and entity type.
1. Confirm MVP entity list.
1. Confirm lifecycle search behavior.
1. Confirm minimum required fields for AI projection.
1. Confirm whether unresolved relationships are indexed with warnings or blocked.
1. Confirm attribute-level identity strategy.
1. Confirm security metadata required before search rollout.
>
> **Recommended Next Step**
> Schedule a design review focused only on identity, lifecycle, relationship resolution, and classification fields. These decisions must be stable before building the AI chunk builder, vector index, and Neo4j projection.
>
