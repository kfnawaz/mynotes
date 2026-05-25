# Data Compass — Metadata Catalog Data Model

**Project:** In-house Replacement of Atlan Crawlers/Miners
**Document purpose:** Authoritative record of the internal Metadata Catalog DB data model, reconstructed from the data dictionary (50 screenshots, **34 entities**). This is the **target contract** for the normalization layer of the new crawlers/miners.
**Status:** Reconstructed from data dictionary. Field-level details verified against all 50 screenshots. Cross-reference cardinalities should be confirmed against the live DDL.

---

## 1. Overview

The model is a federated **enterprise data catalog** ("Data Compass") of **34 entity types**. It describes physical and logical data assets, their business/semantic meaning, the lineage and contracts that connect them, runtime events, and governance/retention classifications.

### 1.1 Entity Families

| Family | Entities | Crawler relevance |
|---|---|---|
| **Physical & logical assets** | `Distribution`, `Dataset`, `DataModel`, `DataModelEntity`, `DataModelAttribute`, `DataService`, `DataProduct`, `DataDomain` | **Primary crawler output** |
| **Business / semantic** | `BusinessGlossary`, `BusinessGlossaryTerm`, `DataConcept`, `DataElement`, `DataConceptQualifierRole`, `DataConceptQualifierValue`, `BusinessContext`, `BusinessProcess` | Fed by Glossary SoR — not crawler scope |
| **Lineage & movement** | `DataFlow`, `DataProcess`, `DataContract`, `DataOffer` | `DataFlow`/`DataProcess` = **miner output** |
| **Event records** | `DataConsumptionEvent`, `DataProductionEvent`, `DataProcessStartEvent`, `DataProcessCompleteEvent`, `DataQualityMeasurementEvent`, `DataContractBreachEvent`, `CompensatingActionEvent` | **Miner output** (from query/access history) |
| **Governance & classification** | `DataAuthorityDeclaration`, `DataNodeRecordClassification`, `DistributionRecordClassification`, `DataConceptDefaultRecordClassificationCode`, `DataQualityMetric` | Fed by SoR/event systems — limited crawler scope |
| **Foundational** | `Node`, `Report` | `Node` referenced by nearly every entity |

Entity count by family: Physical & logical assets (8), Business/semantic (8), Lineage & movement (4), Event records (7), Governance & classification (5), Foundational (2) = **34 total**.

### 1.2 Attribute Classes

Every entity's fields fall into two visually distinguished classes:

- **Data Compass Managed Attribute** (blue) — platform-owned header/identity fields. Consistent across entities.
- **General Attribute** (green) — entity-specific body fields. These are what a crawler/miner must populate.

---

## 2. The Shared "Managed Item" Base

Nearly every primary entity begins with an identical header block. **This is the normalization anchor for the crawler — emit it once, uniformly, for every asset type.**

| Field | Type | Required | Description | LDM Mapping |
|---|---|---|---|---|
| `guid` | String | Yes (1) | Globally unique identifier ensuring uniqueness within Data Compass | Managed Item:Identifier |
| `version` | Integer | Yes (1) | Version number for tracking changes over time | Managed Item:Identifier |
| `lifecycleStatus` | String (enum) | Yes (1) | Current lifecycle status. Default `DRAFT` | Managed Item:Lifecycle Status |
| `createdTimestamp` | String | Yes (1) | When the item was initially created | Managed Item:Creation Date |
| `createdBy` | String | Yes (1) | Identifier (SID or FID) of the Agent that created the item | Catalog Resource:Created By |
| `updatedTimestamp` | String | No (0..1) | When the item was last updated | Managed Item:Last Update Date |
| `updatedBy` | String | No (0..1) | Identifier of the Agent that last updated the item | Catalog Resource:Updated By |
| `certifiedTimestamp` | String | No (0..1) | When the item was certified | Managed Item:Effective Date |
| `certifiedBy` | String | No (0..1) | Identifier of the Agent that certified the item | Catalog Resource:Certified By |
| `jrn` | String | No (0..1) | JPMorgan Resource Name uniquely identifying the item. Optional at creation | Catalog Resource:JRN |
| `owners[]` | Array | No (0..*) | The worker(s) that own the item | Catalog Resource:Owner |
| `owners[].sid` | String | Yes (1, when section used) | SID identifying the owning worker | Catalog Resource:Owner |

**`lifecycleStatus` enum:** `DRAFT`, `PUBLISHED`, `PENDING_APPROVAL`, `REJECTED`, `APPROVED`, `DEPRECATED`

Some entities (glossary/semantic types, event types) use a reduced header — see per-entity notes in Section 5.

---

## 3. Cross-Cutting Concepts

### 3.1 Identity: `guid` vs `jrn`

- **`guid`** — internal Data Compass uniqueness key.
- **`jrn`** (JPMorgan Resource Name) — the resource identifier used for **all cross-entity references**. Every relationship in the model points via a `*.jrn` field.

> **Critical crawler design decision:** The crawler must construct a **deterministic, stable `jrn`** for every Databricks object so that re-crawls produce identical keys and lineage edges remain intact. Confirm whether the crawler mints `guid`/`jrn` or whether the catalog DB assigns them on load.

### 3.2 The `Node` Reference Pattern

`Node` is a foundational entity. Throughout the model, participants in flows/events/services are referenced as a **node pair**:

- `node.nodeId` (String) — the node identifier
- `node.nodeType` (String, enum) — `EXTERNAL_SOURCE`, `INTELLIGENT_SOLUTION`, `SEAL`, `USER_TOOL`, `OTHER` (default `SEAL`)

This pair appears as `node`, `dataProvider.node`, `dataConsumer.node`, `eventBrokerNode`, `performingNodes[]`, etc. Any crawler/miner emitting provider/consumer relationships must resolve these node references.

### 3.3 Owner Role Enum (governance entities)

Where `owners[].role` is present (`DataModel`, `DataProcess`, `DataService`, `DataOffer`, `Distribution`, `Report`, etc.):
`ADMIN`, `GUEST`, `CT_IA`, `CT_GOV`, `CDO`, `CDO_DELEGATE`, `RO`, `RO_DELEGATE`, `BPO`, `BPO_DELEGATE`, `CTO`, `TOWER_IA`, `DDO`, `DPM`, `DSO`, `DO`, `DE`

### 3.4 Recurring Enumerations

| Enum | Values |
|---|---|
| **Data type** (attributes) | `BOOLEAN, CHAR, STRING, DATE, TIME, TIMESTAMP, INTEGER, LONG, DOUBLE, FLOAT, DECIMAL, UUID, JSON, XML, BYTE, BINARY, ARRAY, OTHER` |
| **Database type** | `CASSANDRA, DATABRICKS, DYNAMODB, MONGODB, MSSQL, MYSQL, ORACLE, POSTGRES, SNOWFLAKE, OTHER` |
| **Entity physical type** | `TABLE, VIEW, MATERIALIZED_VIEW` |
| **Confidentiality classification** | `PUBLIC, INTERNAL, CONFIDENTIAL, HIGHLY_CONFIDENTIAL` |
| **Event type** | `DATA_CONSUMPTION, DATA_PRODUCTION, DATA_PROCESS_EXECUTION, DATA_QUALITY_MEASUREMENT, COMPENSATING_ACTION, DATA_CONTRACT_BREACH` |
| **Execution method** | `MANUAL, AUTOMATED, SEMI_AUTOMATED` (default `AUTOMATED`) |
| **Execution frequency** | `INTRA-DAY, DAILY, TWICE_WEEKLY, WEEKLY, BI_WEEKLY, MONTHLY, BI_MONTHLY, QUARTERLY, SEMI_ANUALLY, ANNUALLY, ADHOC` |

### 3.5 The Data Protection / Classification Block

A reusable nested object appearing on attribute-bearing entities (`DataModelAttribute`, `Dataset`, `Distribution`, `Report`):

`isPersonalInformation`, `isSensitivePersonalInformation`, `isClientConfidentialInformation`, `isMaterialNonPublicInformation`, `isDataSubjectAccessRequestIndicator`, `isPurgeIndicator` (all Boolean), plus `informationConfidentialityClassification` (enum).

---

## 4. Entity Relationship Map

Relationships are expressed via `*.jrn` reference fields. Key structural links:

```
DataDomain
  └─ DataProduct (dataProductDomain)
       ├─ inputDatasets[] / outputDatasets[]            → Dataset
       ├─ inputDataServices[] / outputDataServices[]    → DataService
       └─ inputDataDistributions[] / outputDataDistributions[]  → Distribution  [TACTICAL]

Dataset
  ├─ boundarySets[]                  → DataConcept (+ DataConceptQualifiers)
  ├─ conceptualDataModelSpecification → DataModel
  └─ logicalDataSchemaSpecification  → DataModelAttribute[]

Distribution  (physical asset — PRIMARY CRAWLER TARGET)
  ├─ dataset                         → Dataset
  ├─ physicalSpecification           → entityStoreType / entitySchemaName / dataAttributes[]
  └─ physicalDataModelSpecification  → DataModel / DataModelEntity / DataModelAttribute

DataModel
  ├─ dataModelEntities[]             → DataModelEntity
  └─ dataModelEntityRelationships[]  → source/target DataEntity & DataAttribute

DataModelEntity
  ├─ dataConcepts[]                  → DataConcept
  └─ dataModelAttributes[]           → DataModelAttribute

DataModelAttribute
  └─ dataElements[]                  → DataElement

DataService
  └─ dataDistributions[]             → Distribution
     (interface variants: api / database / kafka / storage)

LINEAGE
DataFlow      : dataProvider → dataConsumer, dataAssets[]      (observed lineage)
DataProcess   : inboundDataDistributions[] → outboundDataAsset, transformations[]
DataContract  : dataProvider → dataConsumer, agreements[]
DataOffer     : target, dataQualityCommitments[], timeliness/completeness/schema commitments

SEMANTIC LAYER
BusinessGlossary → BusinessGlossaryTerm → DataConcept → DataElement
DataConcept ←→ DataConceptQualifierRole → DataConceptQualifierValue

EVENTS (all reference assets / processes / nodes via jrn)
DataConsumptionEvent, DataProductionEvent, DataProcessStartEvent,
DataProcessCompleteEvent, DataQualityMeasurementEvent,
DataContractBreachEvent, CompensatingActionEvent

GOVERNANCE
DataAuthorityDeclaration       → node, businessElementTerms[]
DataNodeRecordClassification   → dataNode, recordRetentionRequirement
DistributionRecordClassification → distribution, recordRetentionRequirement
DataConceptDefaultRecordClassificationCode → dataConcept, recordClassificationCode
DataQualityMetric              → assessedArtifacts[], referencedArtifacts[]
```

---

## 5. Entity Catalog

For each entity: purpose, header type (Full = Managed Item base; Reduced = name/displayName/description only; Event = event header), and the distinctive body fields. Standard header fields (Section 2) are not repeated.

### 5.1 Physical & Logical Assets

#### Distribution — *(Full header + node + owners[].role)*
Physical instantiation of a Dataset. **Primary crawler target for Databricks tables/views.**
Key fields: `format[]` (`AVRO, BINARY, CSV, ICEBERG, JSON, PARQUET, TXT, XML, OTHER`), `dataset` (→Dataset), `physicalSpecification` (`entityStoreType` [database-type enum], `entityStoreName`, `entitySchemaName`, `entityPhysicalType` [`TABLE/VIEW/MATERIALIZED_VIEW`], `entityResourceURI`, `dataAttributes[]` — full attribute spec + data-protection block), `physicalDataModelSpecification` (→DataModel/Entity/Attribute), `geographicScope` (`isGlobal`, `countryGroups[]`, `countries[]`).

#### Dataset — *(Full header)*
Logical data asset. Key fields: `documentDataSpecificationURL`, `boundarySets[]` (→DataConcept + `dataConceptQualifiers[]`), `conceptualDataModelSpecification` (→DataModel), `logicalDataModelSpecification` (→DataModel entities/attributes), `logicalDataSchemaSpecification` (`dataAttributes[]` — name, dataType, multiplicity, position, precision/size/scale, key flags, `dataClassification` block, `dataElements[]`).

#### DataModel — *(Full header + node)*
Key fields: `dataModelType` (`CONCEPTUAL, LOGICAL, PHYSICAL`, default `PHYSICAL`), `databaseType` (database-type enum), `documentaionURL`, `dataModelEntities[]`, `dataModelEntityRelationships[]` (`type`: `ASSOCIATION/GENERALIZATION`, `relationshipName`, source/target DataEntity & DataAttribute with `sourceId`, `targetMultiplicity`).

#### DataModelEntity — *(Full header)*
Key fields: `subjectArea`, `documentaionURL`, `entityPhysicalType` (`TABLE/VIEW/MATERIALIZED_VIEW`), `entityStoreName`, `entitySchemaName`, `dataConcepts[]`, `dataModelAttributes[]`, `entityResourceURI`.

#### DataModelAttribute — *(Full header)*
Key fields: `dataType` (data-type enum), `multiplicity`, `size`, `precision`, `scale`, `position`, key/behavior booleans (`isPrimaryKey`, `isForeignKey`, `isIndexed`, `isGenerated`, `isAutoIncremented`, `isEnumerated`, `isNullable`, `isDeprecated`, `isUnique`), `dataProtection` block, `dataElements[]` (→DataElement).

#### DataService — *(Full header + node + owners[].role)*
Service providing access to a Distribution. Key fields: `interfaceType` (`GENERIC, API, DATABASE, STORAGE`), `type` (`PROVIDER, CONSUMER`), `authenticationType` (`CERTIFICATE_BASED, IDPWD, KERBEROS, ONE_WAY_SSL`), `dataDistributions[]`. Interface-specific sub-objects: `api` (urls[], type `REST/SOAP/GRAPHQL`), `database` (type [database enum], connectionString, objectName, query), `kafka` (brokerLists[], clusterName, topicName, messageKey/ValueType, region), `storage` (type `FILE/OBJECT_STORE`, fileNamePattern, path, resourceURI, region).

#### DataProduct — *(Full header)*
Key fields: `dataProductDomain` (→DataDomain), `outputDatasets[]`, `outputDataServices[]`, `inputDatasets[]`, `inputDataServices[]`, `inputDataDistributions[]` `[TACTICAL]`, `outputDataDistributions[]` `[TACTICAL]`.
> Note: `[TACTICAL]` distribution fields are flagged in the dictionary as supporting the *current Atlan implementation*. **Confirm whether these stay post-migration.**

#### DataDomain — *(Full header)*
Key fields: `parent` (→DataDomain, hierarchical).

### 5.2 Business / Semantic Layer

#### BusinessGlossary — *(Reduced header)*
`name`, `displayName`, `description`, `parent` (→glossary, hierarchical), `type` (`DATA_CONCEPT, DATA_CONCEPT_QUALIFIER, DATA_ELEMENT, OTHER`), `scope` (`FIRMWIDE, LOB, SUB_LOB, PERSONAL`), `businessUnitNode.buNodeId`.

#### BusinessGlossaryTerm — *(Reduced header)*
`name`, `displayName`, `description`, `businessGlossary` (→glossary), `parent` (hierarchical), `alternateIdentifiers[]` (`sourceName`, `sourceId`).

#### DataConcept — *(Reduced header)*
Same shape as BusinessGlossaryTerm plus `mandatoryQualifierRoles[]` (→DataConceptQualifierRole).

#### DataElement — *(Reduced header)*
Term residing in a DataConcept. `name`, `displayName`, `description`, `businessGlossary`, `parent`, `alternateIdentifiers[]`, `dataConcept` (→DataConcept). LDM mapping: *Business Element Term*.

#### DataConceptQualifierRole — *(Reduced header)*
`name`, `displayName`, `description`, `businessGlossary`, `parent`, `alternateIdentifiers[]`, `qualifierGroup` (glossary of qualifier values).

#### DataConceptQualifierValue — *(Reduced header)*
`name`, `displayName`, `description`, `businessGlossary`, `parent`, `alternateIdentifiers[]`.

#### BusinessContext — *(Full header)*
Key fields: `catalogArtifact` (subject of the context), `dataTieringDesignation` (`cdo` [large org enum], `level` `LEVEL_1/LEVEL_2`, `category`, `subCategory`), `keyDataElementDeclarations[]` (`type` `ENDPOINT/INPUT`, `criteria`, `attribute`), `regulationScopeDeclarations[]` (`regulation`, e.g. BCBS239).

#### BusinessProcess — *(Full header)*
Key fields: `businessProcessType` (`BUSINESS_PROCESS, SUB_PROCESS`), `parent`, `sequenceNumber`, `performingNodes[]` (nodeId/nodeType), `executionFrequency`, `businessUnitLevel1/2/3` (id+name), `firmwideIndicator`, `inboundDataDistributions[]`, `outboundDataAssets[]`.

### 5.3 Lineage & Movement

#### DataFlow — *(Full header)* — **MINER OUTPUT**
Observed lineage. Key fields: `lastObservedTimestamp`, `lastObservedEventId`, `observedDataContract`, `precision` (`CONCEPTUAL, FUNCTIONAL, TECHNICAL`), `dataProvider` (node + functionalId + dataService), `dataConsumer` (same), `dataAssets[]` (transferred assets).

#### DataProcess — *(Full header + node + owners[].role)* — **MINER OUTPUT**
Key fields: `processGroup`, `executionFrequency`, `executionMethod`, `performingTeam`, `mlModelIds[]`, `inboundDataDistributions[]`, `outboundDataAsset`, `transformations[]` (`targetAttribute`, `sourceAttributes[]`, `logic`, `types[]`: `AGGREGATION, CONVERSION, DERIVATION, FORMAT_CHANGE, PASSTHROUGH_FIELD_NAME_CHANGE, REPRESENTATION_CONVERSION, OTHER`).

#### DataContract — *(Full header)*
Key fields: `dataProvider` (node + `approvals[]` with role `APPLICATION_OWNER/DATA_OWNER`), `dataConsumer` (same), `agreements[]` (`dataAsset`, `providerDataOffer`, `consumerDataOffer`, `expectation` with `timeliness`, `dataQualityMetrics[]` [`performingNode`, `expectationType` `EXPECTED_RESULT/THRESHOLD`, `thresholds[]` with `comparator`], `dataUsage` [`purpose`, `onwardDistributionPermitted`, `dataUseCouncilReferenceIds[]`]).

#### DataOffer — *(Full header + node + owners[].role)*
Key fields: `target`, `dataQualityCommitments[]` (`controlId`, `dataQualityMetrics[]` with `executionFrequency`), `timelinessCommitment` (`publicationFrequency`, `schedule` [`scheduleType` `CRON/GLOBAL_CALENDAR/BUSINESS_DAYS`, `cron`, `days`, `time`, `frequency`, `holidayCalendar`]), `completenessCommitment.methods[]` (`HEADER_FOOTER, CLOSING_TAGS, RECORD_COUNT, CHECKSUM`), `schemaCommitment.dataAttributes[]`, `changeNotificationCommitments[]` (`changeType` `BREAKING/NON_BREAKING`, `value`, `period`), `synchronizationCommitment`.

### 5.4 Event Records — **MINER OUTPUT**

All events share an **Event header**: `guid`, `createdTimestamp`, `createdBy`, `eventId`, `eventType` (event-type enum), `eventTimestamp`, `executionTimestamp`, `executionMethod`, `executionPerformedBy[]`, `businessDate`, `eventBrokerNode` (nodeId/nodeType), `eventCorrelationReference`.

#### DataConsumptionEvent
+ `dataConsumer` (node + functionalId + dataService), `dataAsset` (+instanceId), `payloadInformation` (name, type, version, recordsCount, fieldsCount, `checksum`, `dataAssetInstances[]`), `dataProvider`.

#### DataProductionEvent
+ `dataProvider` (node + dataService), `dataProcess` (+instanceId), `dataConsumer`, `payloadInformation`.

#### DataProcessStartEvent
+ `dataProcess` (+instanceId), `lifecycleStage` (`RE_RUN`).

#### DataProcessCompleteEvent
+ `dataProcess` (+instanceId), `inboundDataDistributions[]`, `outboundDataDistributions[]`, `result` (`PASS, FAIL`).

#### DataQualityMeasurementEvent
+ `node`, `dataQualityMetric`, `dataAsset` (+instanceId), `referencedDataAssets[]`, `attributes[]`, `referencedAttributes[]`, `dataService`, `result`, `resultReason`, `recordsCount`, `passedRecordCount`, `failedRecordCount`, `exceptionsReferences[]`.

#### DataContractBreachEvent
+ `dataContractAgreement`, `dataAsset`, `commitmentType` (`TIMELINESS, DATA_QUALITY`), `triggeringEventIds[]`, `timeliness` (`expectedTimestamp`, `actualTimestamp`).

#### CompensatingActionEvent
+ `triggeringDataEventId`, `actionUndertaken`.

### 5.5 Governance & Classification

#### DataAuthorityDeclaration — *(Full header)*
Key fields: `node`, `dataAuthorityType` (`SYSTEM_OF_CAPTURE, SYSTEM_OF_RECORD, AUTHORITTIVE_DATA_SOURCE`), `businessElementTerms[]`.

#### DataQualityMetric — *(Full header)*
Key fields: `definition`, `dimension` (`ACCURACY, COMPLETENESS, TIMELINESS`), `category` (`BUSINESS_DATA_QUALITY, TECHNICAL_DATA_QUALITY`), `type` (large enum: `STALE_COUNT, RECORD_COUNT, Z_SCORE, ... CHECKSUM, FORMAT, RECONCILIATION`), `resultType` (`PASS_FAIL, BOOLEAN, STRING, INTEGER, PERCENTAGE, PASS_FAIL_WARN`), `level` (`GLOSSARY, LOGICAL, PHYSICAL`), `assessedArtifacts[]`, `referencedArtifacts[]`.

#### DataNodeRecordClassification — *(No standard header — keyed by dataNode)*
`dataNode` (nodeId/nodeType), `dataNodeRecordClassification[]` → `recordClassCode` (`classCode`, `recordType` `OBR/DBU`, `classCodeName`, `legalBusinessCategory`, `containsPersonalInformationIndicator`) + `recordRetentionRequirement` → `countryRetentionRequirement[]` (`scopeType` `COUNTRY/GLOBAL`, `retentionModel` `FLAT/ACTIVE+`, `retentionPeriodValue`, `retentionPeriodTimeUnit` `YEAR/MONTH/DAY`, `retentionEvent`, `exceptionRecordRetentionRequirement[]`).

#### DistributionRecordClassification — *(No standard header — keyed by distribution)*
Same retention structure as above, keyed by `distribution` instead of `dataNode`. `recordRetentionRequirement.retentionPeriodDriver` (`JURISDICTION, SIMPLIFIED`), `simplifiedJustificationRationale`.

#### DataConceptDefaultRecordClassificationCode — *(Full header)*
Maps a default record classification code to a Data Concept.
Key fields: `Guidelines` (additional information regarding the default classification), `dataConcept` (→DataConcept), `recordClassificationCode` (→Record Classification:Record Class Code).

### 5.6 Foundational

#### Node — *(Full header)*
The referenced entity behind every `nodeId`/`nodeType` pair across the model.
Key field: `type` (`EXTERNAL_SOURCE, INTELLIGENT_SOLUTION, SEAL, USER_TOOL, OTHER`, default `SEAL`).

#### Report — *(Full header + node + owners[].role)*
Key fields: `reportIdentificationNumber`, `productionFrequency` (execution-frequency enum), `isRegulatoryReleated`, `reportSpecification.reportAttributes[]` (full attribute spec: `alternateIdentifiers[]`, `scheduleName`, `dataClassification` block, `dataElements[]`, `name`, `dataType`, `multiplicity`, sizing fields, behavior booleans).

---

## 6. Implications for Crawler/Miner Scope (Databricks-First)

| Concern | Detail |
|---|---|
| **Primary crawler outputs** | `Distribution`, `DataModelEntity`, `DataModelAttribute`, `DataService`, `Dataset`, `DataModel` |
| **Miner outputs** | `DataFlow`, `DataProcess`, and the seven Event entities (from Unity Catalog `system.access.*` / query history) |
| **Out of crawler scope** | Semantic layer (Glossary SoR), most Governance/retention entities (SoR + event systems) |
| **Identity-resolution risk** | Deterministic `jrn` construction for every Databricks object — settle before implementation |
| **Node resolution** | Every provider/consumer/broker reference resolves to a `Node`; miner must populate node pairs consistently |
| **`[TACTICAL]` fields** | `DataProduct.input/outputDataDistributions[]` exist to support the current Atlan implementation — confirm post-migration treatment |
| **Header uniformity** | The Section 2 base block is emitted identically for every primary asset — build it once in the normalization layer |

## 7. Open Questions to Confirm Against Live DDL

1. Does the crawler mint `guid` and `jrn`, or does the catalog DB assign them on load?
2. Exact cardinalities and FK directions for cross-entity `*.jrn` references.
3. Post-migration status of `[TACTICAL]` `DataProduct` distribution fields.
4. Soft-delete / tombstone representation — no explicit lifecycle field for *removed* assets was observed beyond `lifecycleStatus = DEPRECATED`. Confirm how the crawler signals an asset that no longer exists at source.
5. Whether `DataNodeRecordClassification` / `DistributionRecordClassification` (headerless entities) are write-targets for any crawler/miner or purely SoR-fed.
