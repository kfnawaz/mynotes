# 06. AI Document and Chunk Builder

## 1. Purpose

The AI Document and Chunk Builder converts structured Data Compass metadata into readable, deterministic, source-backed text records that can be embedded and retrieved by semantic search.

This component is the bridge between the canonical metadata model and the vector retrieval layer. It should not simply serialize MongoDB JSON. It must produce concise, relationship-aware, human-readable descriptions of assets, attributes, services, lineage, processes, reports, classifications, and business mappings.

## 2. Role in the Architecture

```
MongoDB Canonical Metadata
   -> Relationship Resolver
   -> AI Document and Chunk Builder
   -> Embedding Service
   -> MongoDB Atlas Vector Index
   -> Semantic / Hybrid Search
   -> GraphRAG Context Assembly
```

## 3. Design Principles

| Principle | Description |
| --- | --- |
| Source-backed | Every chunk must reference the source guid, jrn, entity type, collection, version, and lifecycle status. |
| Deterministic | Re-running the same source record should produce the same chunk IDs. |
| Relationship-aware | Chunks should include relevant parent, child, upstream, downstream, service, report, and product context. |
| Purpose-specific | Generate different chunk types for summary, attributes, lineage, service, classification, and process transformation. |
| Search-friendly | Text should be written in natural language so semantic search can match business questions. |
| Filterable | Each chunk must include metadata fields needed for security, lifecycle, ownership, and classification filtering. |
| Rebuildable | Chunks are derived projections and can be deleted/rebuilt from canonical metadata. |

## 4. In Scope

- Define standard AI chunk schema.
- Build chunk templates for MVP entities.
- Generate deterministic chunk IDs.
- Add source traceability metadata.
- Add relationship metadata.
- Add classification/security metadata.
- Support batch chunk generation.
- Support re-generation by source jrn, entity type, and changed relationship.
- Produce reviewable sample chunks for stakeholders.

## 5. Out of Scope

- Generating embeddings.
- Persisting vector records.
- Running vector search.
- Generating final LLM answers.
- Updating authoritative metadata.
- Replacing the canonical metadata model.

## 6. MVP Entity Coverage

| Entity | Chunk Types | MVP Priority | Rationale |
| --- | --- | --- | --- |
| Distribution | asset_summary, physical_specification, attribute_rollup, classification_summary, lineage_summary | P0 | Primary physical asset and crawler output. |
| Dataset | asset_summary, logical_schema_summary, business_boundary_summary | P0 | Logical asset layer that connects business and physical metadata. |
| DataModelEntity | entity_summary, attribute_rollup | P0 | Table/view/entity representation. |
| DataModelAttribute | attribute_summary, classification_summary, business_mapping_summary | P0 | Column-level search and governance discovery. |
| DataService | service_summary, interface_summary, exposed_assets_summary | P1 | Explains how assets are exposed or consumed. |
| DataFlow | lineage_summary | P1 | Observed provider-consumer lineage. |
| DataProcess | process_summary, transformation_summary | P1 | Process and transformation lineage. |
| Node | node_summary, related_asset_summary | P1 | Foundational participant in flows and services. |
| Report | report_summary, report_attribute_summary, dependency_summary | P1 | Downstream consumption and reporting impact analysis. |

## 7. Standard AI Chunk Schema

```json
{
  "chunk_id": "<deterministic id>",
  "chunk_type": "asset_summary | attribute_summary | lineage_summary | service_summary | classification_summary | process_transformation_summary | business_mapping_summary",
  "text": "Human-readable AI-ready text.",
  "source": {
    "guid": "<source guid>",
    "jrn": "<source jrn>",
    "collection": "Distribution",
    "entity_type": "Distribution",
    "version": 1,
    "lifecycleStatus": "PUBLISHED",
    "createdTimestamp": "",
    "updatedTimestamp": ""
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
    "isPurgeCandidate": false,
    "sourcePlatform": "Databricks"
  },
  "relationships": {
    "datasetJrn": "",
    "distributionJrns": [],
    "dataServiceJrns": [],
    "dataProductJrns": [],
    "dataDomainJrns": [],
    "upstreamJrns": [],
    "downstreamJrns": [],
    "reportJrns": [],
    "attributeJrns": [],
    "dataElementJrns": []
  },
  "chunking": {
    "schemaVersion": 1,
    "templateVersion": 1,
    "generatedAt": "",
    "generatedBy": "data-compass-ai-indexer"
  }
}
```

## 8. Chunk ID Pattern

Recommended pattern:

```
<source_jrn>::<entity_type>::<chunk_type>::<sequence>::v<chunk_schema_version>
```

Example:

```
jrn:databricks:workspace:catalog:schema:customer_transactions::Distribution::asset_summary::001::v1
```

### Rules

- Chunk ID must be stable across repeated runs when the source has not changed.
- Chunk ID should not include volatile timestamps.
- Chunk ID should include chunk type and sequence.
- If jrn is missing, use a quarantined fallback ID and mark the record as not production-indexable.

## 9. Chunk Template Standards

Each template should follow this pattern:

```
Entity Type: <entity type>.
Name: <name/displayName>.
JRN: <jrn>.
Lifecycle Status: <status>.
Summary: <human-readable explanation>.
Relationships: <key related assets>.
Ownership: <owners and roles>.
Classification: <confidentiality and data protection summary>.
Source: <collection and version>.
```

## 10. Distribution Chunk Template

### asset_summary

```
Distribution <name> is a physical data asset stored in <entityStoreType> store <entityStoreName>, schema <entitySchemaName>, as a <entityPhysicalType>. It implements or represents Dataset <dataset name/jrn> when available. The asset has lifecycle status <status>, version <version>, and is owned by <owners>. It contains <attribute count> attributes including <top attributes>. Classification summary: <classification>. Related services: <services>. Upstream assets: <upstream>. Downstream assets: <downstream>.
```

### physical_specification

Include:

- entityStoreType
- entityStoreName
- entitySchemaName
- entityPhysicalType
- entityResourceURI
- format
- geographicScope
- node references

### attribute_rollup

Summarize:

- Number of attributes
- Primary keys
- Foreign keys
- Nullable/deprecated fields
- Sensitive fields
- Business mapped fields

## 11. Dataset Chunk Template

### asset_summary

```
Dataset <name> is a logical data asset in Data Compass. It is associated with conceptual model <model>, logical schema <schema>, and physical distributions <distribution list>. Boundary sets include <data concepts and qualifiers>. The dataset lifecycle status is <status>. Owners: <owners>.
```

### logical_schema_summary

Include:

- logicalDataSchemaSpecification
- logical attributes
- mapped DataElements
- key fields
- classification rollup

## 12. DataModelEntity Chunk Template

```
DataModelEntity <name> represents a <entityPhysicalType> in store <entityStoreName> and schema <entitySchemaName>. Subject area: <subjectArea>. It contains attributes <attribute list>. Related data concepts: <concepts>. Resource URI: <entityResourceURI>.
```

## 13. DataModelAttribute Chunk Template

```
Attribute <name> belongs to <parent entity/distribution>. Data type: <dataType>. Position: <position>. Primary key: <true/false>. Foreign key: <true/false>. Nullable: <true/false>. Unique: <true/false>. Deprecated: <true/false>. Classification: <classification flags>. Business mapping: <DataElement/DataConcept>.
```

### Attribute Chunk Metadata

| Field | Use |
| --- | --- |
| attributeName | Exact field matching. |
| parentJrn | Parent asset filtering. |
| dataType | Field type search. |
| isPrimaryKey | Key discovery. |
| isForeignKey | Relationship discovery. |
| isNullable | Quality/governance search. |
| hasPII / hasSPI | Security and governance filtering. |

## 14. DataService Chunk Template

```
DataService <name> is a <type> service with interface type <interfaceType>. Authentication type is <authenticationType>. It exposes or consumes distributions <distribution list>. API/database/kafka/storage interface details: <safe summary>. Node: <nodeId/nodeType>. Owners: <owners>.
```

## 15. DataFlow Chunk Template

```
DataFlow <name> represents observed lineage from provider <provider node/service> to consumer <consumer node/service>. Transferred data assets: <assets>. Precision: <precision>. Last observed timestamp: <lastObservedTimestamp>. Last observed event ID: <eventId>.
```

## 16. DataProcess Chunk Template

```
DataProcess <name> belongs to process group <processGroup>. It runs with frequency <executionFrequency> using <executionMethod> execution. Inbound distributions: <inbound>. Outbound asset: <outbound>. Transformations: <target attributes, source attributes, logic, transformation types>. Performing team: <team>. Node: <node>.
```

## 17. Report Chunk Template

```
Report <name> has report identification number <reportIdentificationNumber> and production frequency <productionFrequency>. Regulatory related: <true/false>. Node: <node>. Report attributes include <attributes>. Upstream/source assets: <dependencies>.
```

## 18. Relationship-Aware Chunking Rules

| Source Change | Chunks to Regenerate |
| --- | --- |
| Distribution changed | Distribution chunks, related Dataset rollup, related service chunks, graph candidate. |
| Attribute changed | Attribute chunk, parent Distribution rollup, parent DataModelEntity rollup. |
| Dataset changed | Dataset chunk, related Distribution summaries, DataProduct rollup if in scope. |
| DataFlow changed | DataFlow lineage chunk, upstream/downstream relationship chunks. |
| DataProcess changed | Process chunk, transformation chunks, lineage summary chunks. |
| Node changed | Node chunk, related service/process/flow chunks. |
| Report changed | Report chunk and dependency summary. |

## 19. Chunk Size Guidelines

| Chunk Type | Target Size | Notes |
| --- | --- | --- |
| asset_summary | 300-700 words | Should be independently understandable. |
| attribute_summary | 80-200 words | One attribute per chunk where possible. |
| lineage_summary | 150-400 words | Include provider, consumer, asset, precision. |
| service_summary | 150-350 words | Do not expose secrets or credentials. |
| transformation_summary | 200-600 words | Split long transformation lists. |
| classification_summary | 100-300 words | Include classification flags and retention summary. |

## 20. Security Rules

- Do not include secrets, passwords, credentials, connection strings, tokens, or certificates in chunk text.
- Do not place sensitive values in logs.
- Include classification metadata for filtering, but avoid exposing sensitive data values.
- Access filtering is enforced by Search Service, but chunk metadata must provide the fields needed for filtering.
- If a chunk includes restricted metadata, mark it clearly in metadata.

## 21. Quality Checks

| Check | Expected Result |
| --- | --- |
| Source Traceability | Every chunk has source guid and jrn. |
| Deterministic ID | Same input produces same chunk ID. |
| Readability | Text is understandable without raw JSON. |
| Relationship Context | Chunk includes key parent/child/lineage references. |
| Filter Metadata | Chunk includes lifecycle, owner, node, classification. |
| No Secrets | Chunk text excludes credentials and secrets. |
| Size Boundaries | Chunk is within size guidelines. |

## 22. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| CHK-001 | Define AI chunk schema | P0 |
| CHK-002 | Define deterministic chunk ID format | P0 |
| CHK-003 | Build Distribution chunk templates | P0 |
| CHK-004 | Build Dataset chunk templates | P0 |
| CHK-005 | Build DataModelAttribute chunk templates | P0 |
| CHK-006 | Build DataModelEntity chunk templates | P0 |
| CHK-007 | Build DataService chunk templates | P1 |
| CHK-008 | Build DataFlow chunk templates | P1 |
| CHK-009 | Build DataProcess chunk templates | P1 |
| CHK-010 | Build Report chunk templates | P1 |
| CHK-011 | Build chunk generation batch job | P0 |
| CHK-012 | Build sample chunk review report | P1 |
| CHK-013 | Add chunk quality validation | P1 |

## 23. Acceptance Criteria

- Given a valid Distribution, the system creates an asset_summary chunk with source guid, source jrn, lifecycle status, owners, node, physical specification, and related Dataset.
- Given an Attribute, the system creates an attribute_summary chunk with parent context, data type, key flags, and classification metadata.
- Given a DataFlow, the system creates a lineage_summary chunk with provider, consumer, data assets, precision, and last observed timestamp.
- Re-running chunk generation for unchanged input produces the same chunk IDs.
- Deprecated records are either excluded or marked based on lifecycle rules.
- Generated chunks pass quality validation.
- Sample chunks are reviewed by stakeholders before bulk embedding.

## 24. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-030 | Define AI Chunk Schema |
| DC-AI-031 | Build Distribution Summary Chunk Template |
| DC-AI-032 | Build Dataset Summary Chunk Template |
| DC-AI-033 | Build Attribute-Level Chunk Template |
| DC-AI-034 | Build DataService Chunk Template |
| DC-AI-035 | Build DataFlow Lineage Chunk Template |
| DC-AI-036 | Build DataProcess Transformation Chunk Template |
| DC-AI-037 | Build Report Summary Chunk Template |
| DC-AI-038 | Build Chunk Generation Batch Job |
| DC-AI-039 | Build Chunk Quality Review Samples |

## 25. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-CHK-001 | What fields should be excluded from chunk text due to sensitivity? | Security filtering and prompt safety. |
| OQ-CHK-002 | Should long attribute lists be summarized or split into multiple rollup chunks? | Retrieval precision and cost. |
| OQ-CHK-003 | How much upstream/downstream context should be included in asset summaries? | Chunk size and result relevance. |
| OQ-CHK-004 | Should DRAFT assets be chunked for restricted users? | Lifecycle and entitlement design. |
