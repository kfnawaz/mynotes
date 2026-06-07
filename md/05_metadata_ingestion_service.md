# Metadata Ingestion Service

## 1. Purpose

The Metadata Ingestion Service is the controlled entry point for moving Data Compass metadata from the authoritative MongoDB collections into downstream AI and graph projections.

Its job is to read the selected Data Compass metadata entities, validate them, resolve their relationships, identify records that changed, and produce clean ingestion work items for the AI Chunk Builder and Neo4j Graph Builder.

This service does **not** generate AI answers. It prepares trustworthy metadata so that vector search, graph traversal, GraphRAG, and context-plane agents can operate on clean, source-backed data.

## 2. Role in the Target Architecture

```
MongoDB Data Compass Collections
        ↓
Metadata Ingestion Service
        ↓
Validated Canonical Entity Envelope
        ↓
Relationship Resolution Output
        ↓
Indexing Candidate Queue / Collection
        ↓
AI Chunk Builder          Graph Projection Builder
        ↓                         ↓
Vector Index               Neo4j Graph
```

### Key Architecture Principle

MongoDB remains the authoritative source of metadata. The Metadata Ingestion Service produces derived ingestion outputs that are rebuildable from MongoDB source records.

## 3. In Scope

- Read selected MVP entity collections from MongoDB.
- Support batch ingestion by entity type.
- Support ingestion by source **jrn** or **guid**.
- Validate required canonical fields.
- Validate lifecycle and version fields.
- Resolve **jrn**-based relationships.
- Resolve **Node** references using **nodeId** and **nodeType**.
- Normalize owner and owner-role metadata.
- Preserve classification and data-protection metadata.
- Detect changed records using **version** and/or **updatedTimestamp**.
- Produce relationship-aware reindex candidates.
- Track ingestion run status.
- Track source-to-vector and source-to-graph sync status.
- Capture validation failures, relationship-resolution failures, and retryable errors.
- Provide enough operational evidence to support reconciliation and troubleshooting.

## 4. Out of Scope

- Generating embeddings.
- Running vector search.
- Running Neo4j lineage queries.
- Creating LLM answers.
- Performing autonomous metadata enrichment.
- Updating authoritative metadata without an approved write-back workflow.
- Replacing source crawlers/miners.
- Managing user feedback or historical Q&A.

## 5. MVP Entity Scope

The MVP ingestion scope should focus on the entities needed to support asset search, column/attribute search, service exposure search, and basic lineage/impact analysis.

| Entity | MVP Priority | Why It Is Included | Primary Downstream Consumer |
| --- | --- | --- | --- |
| **Distribution** | P0 | Primary physical data asset; critical for table/view search and physical lineage | Chunk Builder, Graph Builder, Search Service |
| **Dataset** | P0 | Logical data asset; connects business/logical meaning to physical assets | Chunk Builder, Graph Builder |
| **DataModelEntity** | P0 | Entity/table/view representation; useful for structure and graph relationships | Chunk Builder, Graph Builder |
| **DataModelAttribute** | P0 | Attribute/column-level search and classification | Chunk Builder, Graph Builder, Security filters |
| **DataService** | P1 | Describes service/interface access to distributions | Chunk Builder, Graph Builder |
| **DataFlow** | P1 | Observed provider-consumer lineage | Chunk Builder, Graph Builder, GraphRAG |
| **DataProcess** | P1 | Process-level consumption, production, and transformations | Chunk Builder, Graph Builder, GraphRAG |
| **Node** | P0 | Foundational entity used across provider/consumer/broker references | Relationship Resolver, Security filters |
| **Report** | P1 | Enables downstream report dependency search | Chunk Builder, Graph Builder |

### Deferred Entities

The following entities can be added after the MVP foundation is stable:

- **DataProduct** and **DataDomain** for product/domain navigation.
- Business glossary and semantic entities for business meaning.
- Governance and retention entities for policy-aware search.
- Event records after summary strategy is defined.

## 6. Primary Source Inputs

| Source Collection / Entity | Input Type | Required for MVP? | Notes |
| --- | --- | --- | --- |
| Distribution | MongoDB collection | Yes | Primary physical asset; often drives downstream indexing |
| Dataset | MongoDB collection | Yes | Logical asset linked to physical distributions |
| DataModelEntity | MongoDB collection | Yes | Structural entity/table/view representation |
| DataModelAttribute | MongoDB collection | Yes | Attribute/column-level metadata |
| DataService | MongoDB collection | Yes, P1 | Data access and exposure metadata |
| DataFlow | MongoDB collection | Yes, P1 | Observed lineage relationship |
| DataProcess | MongoDB collection | Yes, P1 | Process and transformation lineage |
| Node | MongoDB collection | Yes | Required for node pair resolution |
| Report | MongoDB collection | Yes, P1 | Enables report dependency use cases |
| Indexing status collection | MongoDB collection | Yes | Tracks ingestion/vector/graph sync state |
| Failed records collection | MongoDB collection | Yes | Tracks validation and ingestion failures |

## 7. Primary Outputs

| Output | Description | Consumer |
| --- | --- | --- |
| Canonical Entity Envelope | Standard wrapper around each source entity with identity, lifecycle, source metadata, and payload | Chunk Builder, Graph Builder |
| Relationship Resolution Result | Resolved links to related entities by **jrn** and/or node references | Chunk Builder, Graph Builder |
| Indexing Candidate | Work item indicating what must be chunked, embedded, or projected into graph | Chunk Builder, Embedding Service, Graph Builder |
| Validation Failure Record | Record of source entity failing required checks | Operations Dashboard |
| Relationship Failure Record | Missing or unresolved relationship reference | Operations Dashboard, Data Quality Review |
| Ingestion Run Summary | Counts, durations, failures, and completion status for a run | Operations Dashboard |
| Reindex Impact List | Derived list of related assets impacted by a source change | Reindex Orchestrator |

## 8. End-to-End Ingestion Flow

### 8.1 Batch Ingestion Flow

```
1. Start ingestion run.
2. Load ingestion configuration.
3. Select entity types to process.
4. Read records from MongoDB in batches.
5. Wrap each record in Canonical Entity Envelope.
6. Validate required fields.
7. Resolve direct and indirect relationships.
8. Resolve Node references.
9. Normalize owners and classification metadata.
10. Compare source version/timestamp to indexing status.
11. Produce indexing candidates for changed records.
12. Write run metrics and failure records.
13. Mark ingestion run as completed, completed_with_errors, or failed.
```

### 8.2 Incremental Ingestion Flow

```
1. Identify records where updatedTimestamp or version changed since last successful ingestion.
2. Validate changed records.
3. Resolve relationships for changed records.
4. Determine related chunks and graph nodes impacted by the change.
5. Produce incremental indexing candidates.
6. Update ingestion and indexing status.
```

### 8.3 Ad Hoc Reingestion Flow

```
1. User/operator submits source_jrn, source_guid, or entity_type.
2. Ingestion Service reads matching source record(s).
3. Validation and relationship resolution are rerun.
4. Reindex candidates are generated.
5. Downstream chunk/vector/graph jobs are triggered or queued.
```

## 9. Canonical Entity Envelope

Every ingested source entity should be wrapped in a standard envelope before any downstream projection.

```json
{
  "ingestionRunId": "ing-run-2026-06-06-001",
  "source": {
    "collection": "Distribution",
    "guid": "guid-123",
    "jrn": "jrn:datacompass:distribution:example",
    "entityType": "Distribution",
    "version": 7,
    "lifecycleStatus": "PUBLISHED",
    "createdTimestamp": "2026-06-01T10:00:00Z",
    "updatedTimestamp": "2026-06-06T10:00:00Z"
  },
  "identity": {
    "stableKey": "jrn:datacompass:distribution:example",
    "fallbackKey": "guid-123",
    "identityQuality": "JRN_PRESENT"
  },
  "payload": {
    "originalDocument": {}
  },
  "normalized": {
    "name": "example_asset",
    "displayName": "Example Asset",
    "owners": [],
    "node": {
      "nodeId": "SEAL-123",
      "nodeType": "SEAL"
    },
    "classification": {}
  },
  "validation": {
    "status": "VALID",
    "errors": [],
    "warnings": []
  }
}
```

### Envelope Rules

- The envelope must preserve the original source document.
- The envelope must expose standard identity fields.
- The envelope must include validation status.
- The envelope must be usable by both the Chunk Builder and Graph Builder.
- The envelope should not mutate authoritative source metadata.

## 10. Indexing Candidate Contract

Indexing candidates tell downstream services which records need to be processed.

```json
{
  "candidateId": "cand-001",
  "ingestionRunId": "ing-run-2026-06-06-001",
  "sourceGuid": "guid-123",
  "sourceJrn": "jrn:datacompass:distribution:example",
  "entityType": "Distribution",
  "sourceVersion": 7,
  "sourceUpdatedTimestamp": "2026-06-06T10:00:00Z",
  "requiredActions": [
    "BUILD_AI_CHUNKS",
    "UPSERT_VECTOR_INDEX",
    "UPSERT_GRAPH_PROJECTION"
  ],
  "reason": "SOURCE_CHANGED",
  "impactedRelatedJrns": [
    "jrn:datacompass:dataset:example_dataset"
  ],
  "priority": "P0",
  "status": "READY"
}
```

## 11. Indexing Status Contract

The status collection is required for operational trust. It should track source, chunk, vector, and graph sync states.

```json
{
  "sourceGuid": "guid-123",
  "sourceJrn": "jrn:datacompass:distribution:example",
  "entityType": "Distribution",
  "sourceVersion": 7,
  "sourceUpdatedTimestamp": "2026-06-06T10:00:00Z",
  "lastIngestedAt": "2026-06-06T10:05:00Z",
  "lastValidatedAt": "2026-06-06T10:05:01Z",
  "validationStatus": "VALID",
  "chunkStatus": "PENDING",
  "vectorStatus": "NOT_STARTED",
  "graphStatus": "NOT_STARTED",
  "lastSuccessfulVectorVersion": 6,
  "lastSuccessfulGraphVersion": 6,
  "retryCount": 0,
  "lastFailureReason": null,
  "lastFailureStage": null
}
```

## 12. Validation Rules

### 12.1 Common Validation Rules

| Rule ID | Rule | Severity | Applies To |
| --- | --- | --- | --- |
| VAL-COM-001 | **guid** must be present | Error | All managed entities |
| VAL-COM-002 | **version** must be present and numeric | Error | Full-header entities |
| VAL-COM-003 | **lifecycleStatus** must be valid enum value | Error | Full-header entities |
| VAL-COM-004 | **createdTimestamp** must be present | Error | Full-header entities |
| VAL-COM-005 | **createdBy** must be present | Warning/Error based on entity | Full-header entities |
| VAL-COM-006 | **jrn** should be present for relationship/search/graph use | Error for MVP projected entities unless fallback approved | MVP entities |
| VAL-COM-007 | **owners[].sid** should be normalized when owners exist | Warning | Owner-bearing entities |
| VAL-COM-008 | Deprecated/rejected lifecycle statuses must be flagged for default exclusion | Warning | All searchable entities |

### 12.2 MVP Entity Validation Rules

| Entity | Required Fields / Sections | Validation Notes |
| --- | --- | --- |
| Distribution | guid, version, lifecycleStatus, physicalSpecification, physicalSpecification.entityStoreType, physicalSpecification.entityStoreName, physicalSpecification.entitySchemaName, physicalSpecification.entityPhysicalType | Must be sufficient to create asset summary and graph node |
| Dataset | guid, version, lifecycleStatus, name/displayName where available | Must resolve logical asset identity |
| DataModelEntity | guid, version, lifecycleStatus, entityPhysicalType, entityStoreName, entitySchemaName where available | Used for table/view/entity graph |
| DataModelAttribute | guid, version, lifecycleStatus, name, dataType where available | Missing dataType should be warning unless mandatory by source |
| DataService | guid, version, lifecycleStatus, interfaceType, type, dataDistributions[] where available | Linked distributions should be resolved where possible |
| DataFlow | guid, version, lifecycleStatus, dataProvider, dataConsumer, dataAssets[] where available | Provider/consumer node pairs must be validated |
| DataProcess | guid, version, lifecycleStatus, inboundDataDistributions[], outboundDataAsset where available | Transformation metadata should be preserved |
| Node | guid, version, lifecycleStatus, type | Node identity must support nodeId/nodeType resolution |
| Report | guid, version, lifecycleStatus, reportIdentificationNumber where available | Report dependencies may be sparse in MVP |

## 13. Relationship Resolution

The ingestion service must resolve relationships before downstream AI or graph projection. Relationship failures must not be silently ignored.

### 13.1 Resolution Strategy

```
1. Extract relationship references from source entity.
2. Normalize referenced JRN or node pair.
3. Look up target entity by JRN or node pair.
4. Record resolved relationship if target exists.
5. Record unresolved relationship if target is missing.
6. Decide whether unresolved relationship blocks downstream processing or becomes a warning.
```

### 13.2 Relationship Resolution Matrix

| Source Entity | Relationship / Field | Target Entity | MVP Priority | Failure Behavior |
| --- | --- | --- | --- | --- |
| Distribution | dataset | Dataset | P0 | Warning if missing; continue with asset chunk |
| Distribution | physicalDataModelSpecification | DataModel / DataModelEntity / DataModelAttribute | P1 | Warning; continue |
| Dataset | conceptualDataModelSpecification | DataModel | P1 | Warning; continue |
| Dataset | logicalDataSchemaSpecification.dataAttributes[] | DataModelAttribute / embedded attributes | P0 | Warning; include embedded attributes if present |
| DataService | dataDistributions[] | Distribution | P1 | Warning; service chunk can still be generated |
| DataFlow | dataProvider.node | Node | P1 | Warning/Error depending on lineage use case |
| DataFlow | dataConsumer.node | Node | P1 | Warning/Error depending on lineage use case |
| DataFlow | dataAssets[] | Dataset / Distribution / DataAsset | P1 | Warning; lineage edge may be partial |
| DataProcess | inboundDataDistributions[] | Distribution | P1 | Warning; partial graph edge |
| DataProcess | outboundDataAsset | Dataset / Distribution / DataAsset | P1 | Warning; partial graph edge |
| Report | reportSpecification.reportAttributes[] | Attribute-like embedded structures | P1 | Warning; report chunk can still be generated |

### 13.3 Resolution Output Example

```json
{
  "sourceJrn": "jrn:datacompass:distribution:example",
  "relationships": [
    {
      "type": "IMPLEMENTS",
      "targetJrn": "jrn:datacompass:dataset:example_dataset",
      "targetEntityType": "Dataset",
      "resolutionStatus": "RESOLVED"
    },
    {
      "type": "EXPOSED_BY",
      "targetJrn": "jrn:datacompass:service:example_service",
      "targetEntityType": "DataService",
      "resolutionStatus": "UNRESOLVED",
      "reason": "TARGET_NOT_FOUND"
    }
  ]
}
```

## 14. Node Resolution

The Data Compass model uses a recurring node pair pattern:

- **node.nodeId**
- **node.nodeType**

This appears in asset ownership/context, data providers, data consumers, event brokers, performing nodes, and other model areas.

### Node Resolver Responsibilities

- Normalize node IDs.
- Normalize node types.
- Validate node type against known values.
- Resolve node pair to canonical Node entity where possible.
- Attach resolved node metadata to ingestion envelope.
- Record unresolved node references for data quality review.

### Node Type Values

- EXTERNAL_SOURCE
- INTELLIGENT_SOLUTION
- SEAL
- USER_TOOL
- OTHER

## 15. Owner Normalization

Owner metadata appears across several managed entities. The ingestion service should normalize owner fields into a consistent structure for search, security, and ranking.

### Normalized Owner Shape

```json
{
  "owners": [
    {
      "sid": "S123456",
      "role": "DO",
      "sourceRole": "DO",
      "normalizedRole": "DATA_OWNER"
    }
  ]
}
```

### Owner Handling Rules

- Preserve source owner SID.
- Preserve source owner role.
- Normalize known role values when possible.
- Missing owners should be warning, not fatal for MVP ingestion.
- Owner fields should be indexed as metadata filters for search and access control.

## 16. Lifecycle Handling

The ingestion service must tag source records based on lifecycle status so downstream services can filter correctly.

| lifecycleStatus | Ingestion Behavior | Downstream Default |
| --- | --- | --- |
| DRAFT | Ingest and mark restricted | Not searchable by default |
| PUBLISHED | Ingest normally | Searchable |
| PENDING_APPROVAL | Ingest and mark restricted | Searchable only for authorized users |
| REJECTED | Ingest status only or skip projection | Not searchable |
| APPROVED | Ingest normally | Searchable |
| DEPRECATED | Ingest and mark deprecated | Excluded by default; available only for historical mode |

## 17. Classification and Data Protection Handling

The ingestion service must preserve classification fields because they drive search filtering and AI-context safety.

### Classification Fields to Preserve

- isPersonalInformation
- isSensitivePersonalInformation
- isClientConfidentialInformation
- isMaterialNonPublicInformation
- isDataSubjectAccessRequestIndicator
- isPurgeIndicator
- informationConfidentialityClassification

### Classification Rules

- Classification fields must be copied into normalized metadata where present.
- Attribute-level classification should roll up to parent entity where applicable.
- If any attribute has PII/SPI, parent asset should be marked with aggregate indicators for filtering.
- Missing classification should be treated as unknown, not as safe.

## 18. Change Detection

Change detection determines which records require reprocessing.

### Preferred Change Signals

| Signal | Priority | Notes |
| --- | --- | --- |
| version | P0 | Best signal if reliably incremented on metadata change |
| updatedTimestamp | P0 | Useful for batch incremental scan |
| source hash | P1 | Useful when version/timestamp is unreliable |
| manual reindex flag | P1 | Useful for operator-triggered reprocessing |
| downstream dependency impact | P1 | Required for relationship-aware reindexing |

### Change Detection Rules

- If source version changed, reprocess record.
- If updatedTimestamp changed, reprocess record.
- If relationship target changed, reprocess dependent chunks and graph edges.
- If lifecycleStatus changed to DEPRECATED or REJECTED, deprecate downstream vector/graph projections.
- If classification changed, reprocess security metadata and affected chunks.

## 19. Relationship-Aware Reindexing

A change in one record may require reindexing related records.

| Changed Entity | Direct Reindex | Related Reindex Candidates | Rationale |
| --- | --- | --- | --- |
| Distribution | Distribution chunks and graph node | Dataset summary, DataService chunks, DataFlow/DataProcess lineage chunks | Physical asset details affect related context |
| Dataset | Dataset chunks and graph node | Distribution chunks, DataProduct summary when added | Logical meaning affects physical asset summaries |
| DataModelAttribute | Attribute chunks and graph node | Parent Distribution/DataModelEntity chunks | Attribute/classification changes affect asset summaries |
| DataService | Service chunks and graph node | Distribution service summary | Exposure/access context changes |
| DataFlow | Lineage chunks and graph edges | Provider/consumer asset lineage summaries | Observed lineage changes |
| DataProcess | Process chunks and graph edges | Inbound/outbound asset lineage summaries | Transformation context changes |
| Node | Node metadata | Assets/processes/events referencing node | Node context and access filters may change |
| Report | Report chunks and graph node | Downstream dependency summaries | Report dependency search changes |

## 20. Processing Modes

| Mode | Purpose | Trigger | MVP? |
| --- | --- | --- | --- |
| Full Batch | Process all MVP entities | Manual/scheduled | Yes |
| Incremental Batch | Process changed records only | Schedule | Yes |
| Single Asset Reingestion | Reprocess one asset | Manual/API | Yes |
| Failed Record Retry | Retry failed records | Manual/scheduled | Yes |
| Event-Driven Change Stream | Near real-time processing | MongoDB Change Streams | Later phase |
| Backfill | Rebuild after schema/model change | Manual | Yes |

## 21. Error Handling

### Error Categories

| Error Type | Retryable? | Example | Handling |
| --- | --- | --- | --- |
| Source Read Error | Yes | Temporary MongoDB timeout | Retry with backoff |
| Validation Error | No, unless data fixed | Missing required GUID | Persist failure; exclude from downstream |
| Relationship Resolution Warning | No | Missing linked Dataset | Persist warning; continue when allowed |
| Relationship Resolution Error | Depends | Missing critical Node for lineage | Persist failure or partial record based on entity |
| Status Write Error | Yes | Failed to update indexing status | Retry; alert if persistent |
| Configuration Error | No | Unknown collection mapping | Fail run early |

### Failure Record Shape

```json
{
  "failureId": "fail-001",
  "ingestionRunId": "ing-run-2026-06-06-001",
  "sourceGuid": "guid-123",
  "sourceJrn": "jrn:datacompass:distribution:example",
  "entityType": "Distribution",
  "stage": "VALIDATION",
  "severity": "ERROR",
  "code": "MISSING_REQUIRED_FIELD",
  "message": "Required field lifecycleStatus is missing",
  "retryable": false,
  "createdAt": "2026-06-06T10:05:00Z"
}
```

## 22. Security Considerations

The ingestion service does not make final access-control decisions, but it must preserve the metadata needed by the Search Service and GraphRAG layer to enforce access.

### Security Requirements

- Preserve owners and owner roles.
- Preserve node references.
- Preserve lifecycle status.
- Preserve confidentiality classification.
- Preserve PII/SPI indicators.
- Preserve purge and DSAR indicators.
- Mark missing classification as unknown.
- Do not send source payloads to external services during ingestion.
- Avoid logging sensitive field values beyond what is operationally required.

## 23. Observability and Monitoring

### Metrics to Capture

| Metric | Purpose |
| --- | --- |
| records_read_count | Source volume tracking |
| records_valid_count | Data quality tracking |
| records_invalid_count | Data quality tracking |
| relationships_resolved_count | Relationship quality |
| relationships_unresolved_count | Relationship quality |
| indexing_candidates_created_count | Downstream workload |
| run_duration_seconds | Performance |
| failed_records_count | Operational health |
| retry_count | Stability |

### Logs to Capture

- ingestionRunId
- entityType
- sourceGuid
- sourceJrn
- stage
- status
- failure code
- duration
- count summary

## 24. Configuration

The ingestion service should be configuration-driven where possible.

### Configuration Areas

| Configuration | Example |
| --- | --- |
| Entity selection | Distribution, Dataset, DataModelAttribute |
| Batch size | 500 records |
| Lifecycle inclusion | PUBLISHED, APPROVED |
| Deprecated handling | include_status_only, exclude_from_projection |
| Relationship strictness | strict for P0, permissive for P1 |
| Retry policy | max retries, backoff interval |
| Output targets | indexing_candidates, validation_failures, indexing_status |
| Environment | dev, uat, prod |

## 25. APIs / Commands

The service can initially expose internal commands or batch jobs. APIs can be added when operational control is needed.

| Command / Endpoint | Purpose | Phase |
| --- | --- | --- |
| runFullBatch(entityTypes) | Ingest all records for selected entity types | MVP |
| runIncremental(entityTypes, sinceTimestamp) | Ingest changed records | MVP |
| ingestByJrn(sourceJrn) | Reingest one asset and impacted relationships | MVP |
| retryFailed(runId) | Retry failed records | MVP |
| getRunStatus(runId) | Retrieve ingestion run status | MVP |
| getEntityStatus(sourceJrn) | Retrieve source/vector/graph status | MVP |

## 26. Testing Strategy

### Unit Tests

- Entity envelope creation.
- Required field validation.
- Lifecycle status validation.
- Owner normalization.
- Node pair normalization.
- Relationship reference extraction.
- Change detection comparison.

### Integration Tests

- Read sample MongoDB records.
- Validate Distribution/Dataset relationship resolution.
- Validate DataService/Distribution relationship resolution.
- Validate DataFlow provider/consumer resolution.
- Validate indexing candidate creation.
- Validate failure record persistence.

### Data Quality Tests

- Count records by entity type.
- Count missing JRN by entity type.
- Count unresolved relationships by entity type.
- Count missing owners.
- Count missing classification metadata.
- Count deprecated/rejected records.

## 27. Success Criteria

| Category | Success Criteria |
| --- | --- |
| Functional | MVP entity collections can be read in batch mode |
| Functional | Records can be ingested by entity type, source GUID, or source JRN |
| Data Quality | Invalid records are captured with reason and severity |
| Data Quality | Relationship resolution results are persisted |
| Traceability | Every indexing candidate includes sourceGuid, sourceJrn, entityType, version, and lifecycleStatus |
| Security | Owner, node, lifecycle, and classification metadata are preserved |
| Operations | Ingestion run status and counts are visible |
| Operations | Failed records can be reviewed and retried where applicable |
| Reindexing | Changed records produce indexing candidates |
| Reindexing | Related records impacted by a change can be identified |

## 28. Key Pitfalls and Mitigations

| Pitfall | Impact | Mitigation |
| --- | --- | --- |
| Treating ingestion as simple copy | Poor AI/graph quality | Build validation and relationship resolution into ingestion |
| Ignoring relationship failures | Incomplete lineage and weak GraphRAG answers | Persist relationship failure records and quality metrics |
| Missing indexing status | Cannot troubleshoot stale vector/graph data | Build indexing status collection in Phase 1 |
| Assuming missing classification means safe | Security risk | Treat missing classification as unknown |
| Reindexing only changed record | Related chunks remain stale | Build relationship-aware reindex candidate logic |
| Allowing Neo4j/vector stores to become authoritative | Conflicting data sources | Keep derived projections rebuildable from MongoDB |
| Overly strict validation for early MVP | Too many records blocked | Separate blocking errors from warnings |
| No deterministic identity | Duplicate chunks and graph nodes | Use source JRN where available and document fallback rules |

## 29. Jira Stories for Metadata Ingestion Service

### EPIC 3: Metadata Ingestion Service

**Epic Description:** Build the service that reads MongoDB metadata, validates records, resolves relationships, detects changes, and prepares records for chunking and graph projection.

| Jira ID | Story | High-Level Description | Priority |
| --- | --- | --- | --- |
| DC-AI-020 | Build MongoDB Reader for MVP Entities | Read selected Data Compass collections from MongoDB in batch mode with pagination and run-level logging | P0 |
| DC-AI-021 | Build Entity Validator | Validate required fields, lifecycle values, identity fields, and entity-specific requirements | P0 |
| DC-AI-022 | Build JRN Relationship Resolver | Resolve related entities using **jrn** references and persist unresolved relationship warnings/errors | P0 |
| DC-AI-023 | Build Node Resolver | Resolve node references using **nodeId** and **nodeType** and attach node context to ingestion envelope | P0 |
| DC-AI-024 | Build Owner Resolver | Normalize owner SIDs and owner roles for search, ranking, and access filtering | P0 |
| DC-AI-025 | Build Indexing Status Collection | Track ingestion, validation, vector sync, graph sync, failure reason, and retry count | P0 |
| DC-AI-026 | Build Source Change Detection | Detect records requiring reindex using version, updatedTimestamp, and optional source hash | P1 |
| DC-AI-027 | Build Retry and Failure Handling | Persist failed records and support retry of retryable failures | P1 |
| DC-AI-028 | Build Ingestion Run Summary | Capture read/valid/invalid/resolved/unresolved counts and run duration | P1 |
| DC-AI-029 | Build Relationship-Aware Reindex Candidate Generator | Identify related chunks and graph projections affected by a source change | P1 |

### DC-AI-020 — Build MongoDB Reader for MVP Entities

**Objective:** Build a MongoDB reader capable of reading selected MVP entities in controlled batches.

**Acceptance Criteria:**

- Reader connects to configured MongoDB environment.
- Reader supports Distribution, Dataset, DataModelEntity, DataModelAttribute, DataService, DataFlow, DataProcess, Node, and Report.
- Reader supports configurable batch size.
- Reader logs read counts by entity type.
- Reader can run by entity type.
- Reader can read one record by **guid** or **jrn** where supported.

### DC-AI-021 — Build Entity Validator

**Objective:** Validate source records before they become chunking or graph candidates.

**Acceptance Criteria:**

- Validator checks common managed-item fields.
- Validator checks MVP entity-specific required fields.
- Validator distinguishes errors from warnings.
- Invalid records are persisted with failure reason.
- Validation summary is available by ingestion run and entity type.

### DC-AI-022 — Build JRN Relationship Resolver

**Objective:** Resolve relationships between Data Compass entities using **jrn** references.

**Acceptance Criteria:**

- Resolver extracts relationship references from MVP entities.
- Resolver resolves Distribution-to-Dataset links.
- Resolver resolves DataService-to-Distribution links.
- Resolver resolves DataFlow provider/consumer/assets where possible.
- Resolver resolves DataProcess inbound/outbound references where possible.
- Unresolved relationships are persisted with severity and reason.

### DC-AI-023 — Build Node Resolver

**Objective:** Resolve recurring node pair references used across Data Compass metadata.

**Acceptance Criteria:**

- Resolver normalizes **nodeId** and **nodeType**.
- Resolver validates nodeType against known values.
- Resolver resolves node pair to Node entity where possible.
- Missing node references are recorded.
- Resolved node metadata is attached to ingestion envelope.

### DC-AI-024 — Build Owner Resolver

**Objective:** Normalize owner metadata for retrieval filters, ranking, and security.

**Acceptance Criteria:**

- Owner SIDs are extracted and normalized.
- Owner roles are preserved.
- Known owner roles are mapped to normalized role labels.
- Missing owner metadata creates a warning.
- Owner metadata is available in ingestion output.

### DC-AI-025 — Build Indexing Status Collection

**Objective:** Track ingestion and downstream sync status by source entity.

**Acceptance Criteria:**

- Status record includes sourceGuid, sourceJrn, entityType, sourceVersion, and sourceUpdatedTimestamp.
- Status tracks validationStatus, chunkStatus, vectorStatus, and graphStatus.
- Status tracks last success timestamps and versions.
- Status tracks failure reason, failure stage, and retry count.
- Status can be queried by sourceJrn and entityType.

### DC-AI-026 — Build Source Change Detection

**Objective:** Identify records that need reprocessing.

**Acceptance Criteria:**

- Change detection compares source version to last indexed version.
- Change detection compares source updatedTimestamp to last indexed timestamp.
- Changed records produce indexing candidates.
- Lifecycle changes produce appropriate projection actions.
- Classification changes trigger security metadata reprocessing.

### DC-AI-027 — Build Retry and Failure Handling

**Objective:** Persist failures and allow retry of retryable ingestion problems.

**Acceptance Criteria:**

- Failures are stored with runId, sourceGuid, sourceJrn, entityType, stage, code, and message.
- Failures are marked retryable or non-retryable.
- Retry process can rerun retryable failures.
- Retry count is tracked.
- Persistent failures remain visible for operations review.

### DC-AI-028 — Build Ingestion Run Summary

**Objective:** Provide run-level visibility into ingestion processing.

**Acceptance Criteria:**

- Run summary includes start time, end time, status, duration, entity types, and counts.
- Run summary includes read, valid, invalid, warning, resolved, unresolved, and candidate counts.
- Run summary links to failure records.
- Run summary can be used by operations dashboard.

### DC-AI-029 — Build Relationship-Aware Reindex Candidate Generator

**Objective:** Determine downstream chunks and graph records impacted by source changes.

**Acceptance Criteria:**

- Distribution change identifies related Dataset, DataService, DataFlow, and DataProcess impacts.
- Attribute change identifies parent entity/distribution impacts.
- DataFlow/DataProcess change identifies impacted lineage summaries.
- Candidate records include reason and required downstream actions.
- Candidate records are deduplicated by sourceJrn and required action.

## 30. Related Pages

- 03. Scope, MVP, and Phasing
- 04. Canonical / Normalized Metadata Model
- 06. AI Document and Chunk Builder
- 07. Embedding and Vector Indexing
- 08. Graph Projection and Neo4j Builder
- 12. Security and Access Control
- 14. Operations, Monitoring, and Reindexing
- 17. Open Questions and Dependencies
- 18. Jira Backlog Map

## 31. Open Questions

| ID | Question | Impact | Owner | Status |
| --- | --- | --- | --- | --- |
| ING-OQ-001 | Are all MVP entities stored in separate MongoDB collections or mixed collections with type fields? | Affects reader implementation | TBD | Open |
| ING-OQ-002 | Is **version** reliably incremented for all metadata updates? | Affects change detection | TBD | Open |
| ING-OQ-003 | Is **updatedTimestamp** reliable enough for incremental ingestion? | Affects change detection | TBD | Open |
| ING-OQ-004 | What is the approved behavior when **jrn** is missing? | Affects identity and graph projection | TBD | Open |
| ING-OQ-005 | Should unresolved Dataset/Distribution links block projection or only create warnings? | Affects MVP throughput | TBD | Open |
| ING-OQ-006 | Which ownership roles map to search/access-control roles? | Affects security filters | TBD | Open |
| ING-OQ-007 | Should the first MVP use scheduled batch only, or MongoDB Change Streams? | Affects operational complexity | TBD | Open |
| ING-OQ-008 | Which team owns remediation of validation failures? | Affects support model | TBD | Open |
