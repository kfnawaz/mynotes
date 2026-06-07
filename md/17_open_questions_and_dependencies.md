# 17. Open Questions and Dependencies

## 1. Purpose

This page tracks open questions, decisions needed, dependencies, owners, and target dates for Data Compass AI Modernization.

## 2. Open Questions

| ID | Question | Impact | Owner | Needed By | Status |
| --- | --- | --- | --- | --- | --- |
| OQ-001 | Does the crawler mint guid and jrn, or does catalog DB assign them? | Blocks identity design. | TBD | Phase 0 | Open |
| OQ-002 | What is the deterministic jrn format for each source platform? | Blocks chunks and graph nodes. | TBD | Phase 0 | Open |
| OQ-003 | Which lifecycle statuses should be searchable by default? | Blocks retrieval filters. | TBD | Phase 0 | Open |
| OQ-004 | What user roles/entitlements control metadata visibility? | Blocks security design. | TBD | Phase 1 | Open |
| OQ-005 | Should deprecated assets be retained in vector index but filtered, or removed? | Affects reindexing and audit. | TBD | Phase 1 | Open |
| OQ-006 | Which event entities are summarized in MVP? | Affects indexing cost. | TBD | Phase 2 | Open |
| OQ-007 | What are latency targets for exact, semantic, graph, and hybrid search? | Affects architecture and sizing. | TBD | Phase 2 | Open |
| OQ-008 | What source references must be shown in answers and UI? | Affects UX/API. | TBD | Phase 2 | Open |
| OQ-009 | Is MongoDB Atlas Vector Search approved for production use? | Platform dependency. | TBD | Phase 1 | Open |
| OQ-010 | Is Neo4j approved as SaaS, internal managed, or self-managed? | Platform dependency. | TBD | Phase 2 | Open |
| OQ-011 | Which LLM/embedding endpoints are approved for enterprise metadata? | Blocks embedding and answer generation. | TBD | Phase 1 | Open |
| OQ-012 | What data fields are prohibited from chunk text? | Blocks chunk template approval. | TBD | Phase 1 | Open |
| OQ-013 | Who approves AI metadata enrichment suggestions? | Blocks enrichment workflow. | TBD | Phase 5 | Open |
| OQ-014 | What is the target golden question pass rate for release? | Blocks release gate. | TBD | Phase 4 | Open |
| OQ-015 | What tool will host the operations dashboard? | Blocks operations implementation. | TBD | Phase 6 | Open |

## 3. Dependency Register

| ID | Dependency | Type | Impact | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| DEP-001 | MongoDB source collection access | Platform/Data | Required for ingestion. | TBD | Open |
| DEP-002 | Data Compass data model approval | Architecture | Required for canonical contract. | TBD | Open |
| DEP-003 | Embedding model approval | Security/Platform | Required for vector indexing. | TBD | Open |
| DEP-004 | MongoDB Atlas Vector Search environment | Platform | Required for semantic search. | TBD | Open |
| DEP-005 | Neo4j environment | Platform | Required for graph projection. | TBD | Open |
| DEP-006 | Entitlement/access-control source | Security | Required for retrieval filtering. | TBD | Open |
| DEP-007 | Acronym/alias source | Business Context | Required for Context Plane. | TBD | Open |
| DEP-008 | UI integration capacity | Product/UI | Required for user-facing search. | TBD | Open |
| DEP-009 | Observability/logging platform | Operations | Required for dashboards and audit. | TBD | Open |
| DEP-010 | Golden question reviewers | Evaluation | Required for quality gates. | TBD | Open |

## 4. Decision Needed by Phase

### Phase 0

- MVP entity scope.
- guid/jrn identity rules.
- lifecycle filtering rules.
- canonical relationship resolution rules.
- ADR approval process.

### Phase 1

- MongoDB access and collection list.
- validation rules.
- indexing status schema.
- change detection approach.

### Phase 2

- embedding model.
- vector index configuration.
- metadata filters.
- chunk text sensitivity exclusions.

### Phase 3

- Neo4j deployment model.
- graph schema.
- traversal depth defaults.
- graph access-control strategy.

### Phase 4+

- answer generation model.
- citation format.
- confidence/clarification thresholds.
- feedback review process.

## 5. Escalation Criteria

Escalate an open question when:

- It blocks a P0 Jira story.
- It blocks security design.
- It changes architecture direction.
- It affects production approval.
- It impacts data privacy or compliance.
- It delays current sprint commitment.
