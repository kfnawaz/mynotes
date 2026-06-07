# 08. Graph Projection and Neo4j Builder

## 1. Purpose

The Graph Projection and Neo4j Builder converts canonical Data Compass metadata into a Neo4j graph projection that supports lineage traversal, impact analysis, relationship discovery, graph visualization, and GraphRAG.

Neo4j is a derived projection. MongoDB remains the authoritative source of metadata.

## 2. Role in Architecture

```
MongoDB Canonical Metadata
   -> Relationship Resolver
   -> Graph Projection Builder
   -> Neo4j Schema Runner
   -> Neo4j Graph Loaders
   -> Graph Search API
   -> GraphRAG Orchestrator
```

## 3. Design Principles

| Principle | Description |
| --- | --- |
| Rebuildable projection | Neo4j must be rebuildable from MongoDB canonical metadata. |
| JRN as identity | Graph nodes should use jrn as the stable primary identity where available. |
| Relationship-first | Graph should emphasize relationship traversal, not duplicate every source field. |
| Source-backed | Every node and edge must keep source references. |
| Idempotent loads | Re-running loaders should merge/update, not duplicate. |
| Traversal-safe | Queries must support depth limits and access filters. |

## 4. In Scope

- Neo4j node label design.
- Neo4j relationship type design.
- Constraints and indexes.
- Graph loaders for MVP entities.
- Upstream/downstream traversal queries.
- Graph sync status tracking.
- Count reconciliation between MongoDB and Neo4j.

## 5. Out of Scope

- Making Neo4j the source of truth.
- Manual relationship curation for MVP.
- Full graph data science use cases in MVP.
- Full event graph for every runtime event.

## 6. MVP Graph Nodes

| Node Label | Source Entity | Key Properties |
| --- | --- | --- |
| Node | Node | jrn/guid, nodeId, nodeType, name, lifecycleStatus |
| Dataset | Dataset | jrn, guid, name, lifecycleStatus, owners |
| Distribution | Distribution | jrn, guid, name, store, schema, physicalType, lifecycleStatus |
| DataModelEntity | DataModelEntity | jrn, guid, name, subjectArea, physicalType |
| DataModelAttribute | DataModelAttribute | jrn, guid, name, dataType, classification flags |
| DataService | DataService | jrn, guid, name, interfaceType, serviceType |
| DataFlow | DataFlow | jrn, guid, precision, lastObservedTimestamp |
| DataProcess | DataProcess | jrn, guid, processGroup, executionFrequency, executionMethod |
| Report | Report | jrn, guid, reportIdentificationNumber, productionFrequency |
| Owner | owners[].sid | sid, role |

## 7. MVP Relationship Types

| Relationship | From | To | Purpose |
| --- | --- | --- | --- |
| IMPLEMENTS | Distribution | Dataset | Physical implementation of logical asset. |
| HAS_ENTITY | DataModel | DataModelEntity | Model contains entity. |
| CONTAINS_ATTRIBUTE | Distribution/DataModelEntity/Report | DataModelAttribute | Attribute membership. |
| EXPOSES | DataService | Distribution | Service exposes distribution. |
| CONSUMES | DataProduct/DataProcess/Report | Dataset/Distribution | Consumption dependency. |
| PRODUCES | DataProduct/DataProcess | Dataset/Distribution | Production relationship. |
| UPSTREAM_OF | Asset | Asset | Directional lineage. |
| DOWNSTREAM_OF | Asset | Asset | Reverse lineage for traversal convenience. |
| OWNED_BY | Asset | Owner | Ownership. |
| ASSOCIATED_WITH_NODE | Asset/Service/Process/Flow | Node | Node context. |
| MAPS_TO | DataModelAttribute | DataElement | Business mapping, Phase 2. |
| CLASSIFIED_AS | Asset/Attribute | Classification | Governance, Phase 2/3. |

## 8. Graph Node Key Rules

Preferred node key:

```
entity_type + jrn
```

Fallback key when jrn is missing:

```
entity_type + guid
```

Fallback nodes must be marked:

```json
{
  "identityQuality": "FALLBACK_GUID",
  "missingJrn": true
}
```

## 9. Standard Node Properties

All graph nodes should include:

- guid
- jrn
- entityType
- name/displayName
- lifecycleStatus
- version
- sourceCollection
- createdTimestamp
- updatedTimestamp
- identityQuality
- sourcePlatform where available

## 10. Standard Relationship Properties

All relationships should include:

- sourceGuid
- sourceJrn
- sourceCollection
- relationshipType
- observedTimestamp where applicable
- confidence/precision where applicable
- createdAt
- updatedAt
- loaderVersion

## 11. Graph Loader Flow

```
1. Read validated canonical records.
2. Resolve required jrn relationships.
3. Build graph node candidates.
4. Build graph edge candidates.
5. Run schema constraints/indexes.
6. MERGE nodes by stable key.
7. MERGE relationships by stable from/to/type/source.
8. Update graph sync status.
9. Reconcile counts.
```

## 12. Cypher Schema Requirements

Create uniqueness constraints for:

- Distribution.jrn
- Dataset.jrn
- DataModelEntity.jrn
- DataModelAttribute.jrn
- DataService.jrn
- DataFlow.jrn
- DataProcess.jrn
- Report.jrn
- Node.nodeId + Node.nodeType
- Owner.sid

Create indexes for:

- entityType
- lifecycleStatus
- name
- store/schema
- classification flags
- updatedTimestamp

## 13. Loader Responsibilities by Entity

### Distribution / Dataset Loader

- Create Distribution nodes.
- Create Dataset nodes.
- Create IMPLEMENTS relationships.
- Attach owners.
- Attach node relationship if present.

### Attribute Loader

- Create DataModelAttribute nodes.
- Create CONTAINS_ATTRIBUTE relationships.
- Store key flags and classification flags.

### DataService Loader

- Create DataService nodes.
- Create EXPOSES relationships to Distribution.
- Preserve interfaceType and service type.

### DataFlow Loader

- Create DataFlow nodes.
- Resolve provider and consumer.
- Create UPSTREAM_OF and DOWNSTREAM_OF relationships between assets.
- Store precision and last observed timestamp.

### DataProcess Loader

- Create DataProcess nodes.
- Create CONSUMES relationships to inbound assets.
- Create PRODUCES relationships to outbound asset.
- Optionally create transformation relationships for attribute lineage.

### Report Loader

- Create Report nodes.
- Create DEPENDS_ON relationships to assets where available.
- Create CONTAINS_ATTRIBUTE relationships for report attributes.

## 14. Traversal Queries

### Upstream Traversal

Inputs:

- starting jrn
- max depth
- entity type filters
- lifecycle filters
- access filters

Output:

- path
- source asset
- target asset
- relationship types
- relationship metadata
- source references

### Downstream Traversal

Same as upstream, but follows downstream direction.

## 15. GraphRAG Integration

GraphRAG should use Neo4j when:

- User asks upstream/downstream questions.
- User asks impact analysis questions.
- User asks dependency questions.
- Semantic results need relationship expansion.
- Exact asset resolution needs connected context.

## 16. Security Requirements

- Graph traversal must apply lifecycle and access filters.
- Restricted nodes must not be returned to unauthorized users.
- Relationship paths must not leak hidden intermediate nodes.
- Audit logs should capture query, starting node, filters, depth, and returned path IDs.

## 17. Rebuild and Sync Strategy

| Mode | Description |
| --- | --- |
| Full rebuild | Drop/rebuild graph from canonical source. |
| Entity type sync | Rebuild selected entity type. |
| Source jrn sync | Rebuild one asset and affected relationships. |
| Relationship sync | Rebuild edges affected by source relationship change. |

## 18. Reconciliation

Track:

- MongoDB source count by entity type.
- Neo4j node count by label.
- Neo4j relationship count by type.
- Failed graph records.
- Orphan relationship count.
- Missing jrn count.

## 19. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| GRF-001 | Define Neo4j graph schema | P0 |
| GRF-002 | Build Cypher schema runner | P0 |
| GRF-003 | Build Distribution/Dataset loader | P0 |
| GRF-004 | Build Attribute loader | P0 |
| GRF-005 | Build DataService loader | P1 |
| GRF-006 | Build DataFlow loader | P1 |
| GRF-007 | Build DataProcess loader | P1 |
| GRF-008 | Build Report dependency loader | P1 |
| GRF-009 | Build upstream traversal query | P1 |
| GRF-010 | Build downstream traversal query | P1 |
| GRF-011 | Build graph count reconciliation | P1 |

## 20. Acceptance Criteria

- Neo4j schema is deployable idempotently.
- MVP nodes load without duplication.
- Relationships are created using resolved jrn references.
- Upstream and downstream traversal work by source jrn.
- Graph counts can be reconciled against source metadata.
- Missing relationships are logged, not silently ignored.
- Graph can be rebuilt from source metadata.

## 21. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-050 | Define Neo4j Graph Schema |
| DC-AI-051 | Build Cypher Schema Runner |
| DC-AI-052 | Build Distribution/Dataset Graph Loader |
| DC-AI-053 | Build Attribute Graph Loader |
| DC-AI-054 | Build DataService Graph Loader |
| DC-AI-055 | Build DataFlow Graph Loader |
| DC-AI-056 | Build DataProcess Graph Loader |
| DC-AI-057 | Build Report Dependency Graph Loader |
| DC-AI-058 | Build Upstream Traversal Query |
| DC-AI-059 | Build Downstream Traversal Query |

## 22. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-GRF-001 | Is Neo4j SaaS, internal managed, or self-hosted? | Deployment design. |
| OQ-GRF-002 | What depth limit should be default for lineage traversal? | Performance and UX. |
| OQ-GRF-003 | Should reverse DOWNSTREAM_OF edges be stored or derived at query time? | Storage vs query simplicity. |
| OQ-GRF-004 | How should attribute-level transformations be represented in MVP? | Graph complexity. |
