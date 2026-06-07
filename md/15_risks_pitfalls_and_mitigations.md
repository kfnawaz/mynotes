# 15. Risks, Pitfalls, and Mitigations

## 1. Purpose

This page captures major delivery, architecture, data quality, security, operational, and user-trust risks for Data Compass AI Modernization.

## 2. Risk Categories

| Category | Description |
| --- | --- |
| Identity Risk | Problems with guid, jrn, chunk IDs, or graph identity. |
| Relationship Risk | Broken or incomplete cross-entity references. |
| Data Quality Risk | Missing, stale, or incorrect source metadata. |
| Search Quality Risk | Poor semantic results, irrelevant chunks, weak ranking. |
| Graph Risk | Incorrect lineage, duplicated nodes, broken traversal. |
| Security Risk | Unauthorized metadata retrieval or LLM context leakage. |
| Cost/Scale Risk | Excessive embedding, storage, or query cost. |
| Operations Risk | No reindexing, reconciliation, or failure handling. |
| User Trust Risk | Answers are not grounded, cited, or reliable. |
| Delivery Risk | Scope too broad, dependencies unresolved, poor sequencing. |

## 3. Risk Register

| Risk ID | Risk | Category | Impact | Likelihood | Mitigation | Owner | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R-001 | Unstable jrn breaks traceability and graph relationships. | Identity | High | Medium | Define deterministic jrn rules before indexing. | TBD | Open |
| R-002 | Missing jrn prevents relationship resolution. | Identity | High | Medium | Quarantine records and create exception report. | TBD | Open |
| R-003 | Raw JSON embeddings produce poor semantic retrieval. | Search Quality | High | High | Use AI-readable chunk templates. | TBD | Open |
| R-004 | Chunks lack relationship context. | Search Quality | High | Medium | Build relationship-aware chunk generation. | TBD | Open |
| R-005 | Overly large chunks reduce precision. | Search Quality | Medium | Medium | Split by chunk type and purpose. | TBD | Open |
| R-006 | Tiny chunks lack meaning. | Search Quality | Medium | Medium | Include parent asset and source context. | TBD | Open |
| R-007 | Neo4j becomes competing source of truth. | Architecture | High | Medium | Declare Neo4j as rebuildable projection. | TBD | Open |
| R-008 | Graph node duplication due to weak identity. | Graph | High | Medium | Use jrn constraints and idempotent MERGE. | TBD | Open |
| R-009 | Incorrect lineage paths reduce trust. | Graph | High | Medium | Create golden lineage tests and reconciliation. | TBD | Open |
| R-010 | Unauthorized metadata enters LLM context. | Security | Critical | Medium | Enforce filters before context assembly. | TBD | Open |
| R-011 | Restricted graph paths leak hidden assets. | Security | Critical | Medium | Filter nodes/edges before returning paths. | TBD | Open |
| R-012 | Deprecated assets appear in active answers. | Data Quality | High | Medium | Lifecycle filters exclude deprecated by default. | TBD | Open |
| R-013 | Event volume creates high indexing cost and noise. | Cost/Scale | Medium | High | Summarize events instead of embedding all raw events. | TBD | Open |
| R-014 | No reindex controls make fixes slow. | Operations | High | High | Build reindex by jrn and entity type early. | TBD | Open |
| R-015 | No source citations reduce user trust. | User Trust | High | Medium | Require source guid/jrn in all results and answers. | TBD | Open |
| R-016 | Ambiguous asset names cause wrong answers. | User Trust | High | High | Build entity resolver and clarification flow. | TBD | Open |
| R-017 | Metadata enrichment suggestions are accepted without review. | Governance | High | Medium | Require human approval workflow. | TBD | Open |
| R-018 | Search works in demo but fails at scale. | Scale | High | Medium | Measure latency, topK accuracy, and index sizes early. | TBD | Open |
| R-019 | Ownership and node mappings are incomplete. | Security/Data Quality | High | Medium | Validate and report missing owner/node metadata. | TBD | Open |
| R-020 | Teams start building agents before retrieval is reliable. | Delivery | High | High | Sequence delivery: model -> ingestion -> chunks -> vector -> graph -> agents. | TBD | Open |

## 4. Pitfalls by Component

### Canonical Metadata Model

- Treating source-specific metadata as canonical without normalization.
- Not resolving guid vs jrn responsibility.
- Ignoring lifecycle status.
- Failing to define required fields.

### Metadata Ingestion

- Silently skipping invalid records.
- Not tracking relationship resolution failures.
- Not preserving version and updatedTimestamp.
- Not supporting retries.

### Chunk Builder

- Embedding raw JSON.
- Missing source references.
- No relationship context.
- Including credentials or sensitive connection details.

### Vector Indexing

- Not using deterministic upsert.
- Missing metadata filters.
- Not tracking embedding model version.
- Keeping deleted/deprecated chunks active.

### Graph Projection

- Creating duplicate nodes.
- Using names instead of jrn as keys.
- Treating graph as source of truth.
- No depth limits on traversal.

### Search / GraphRAG

- Using vector search for exact lineage questions.
- Not hydrating authoritative source records.
- No citation support.
- Answering despite low confidence or ambiguity.

### Security

- Filtering after LLM context assembly.
- Trusting prompt instructions for security.
- Reusing historical Q&A without entitlement re-check.
- Leaking hidden graph path nodes.

## 5. Mitigation Themes

| Theme | Practices |
| --- | --- |
| Identity Discipline | Deterministic jrn, chunk IDs, graph constraints. |
| Source Traceability | guid/jrn on every chunk, node, edge, result, answer. |
| Retrieval Safety | Security filters before LLM context. |
| Quality Gates | Golden questions and regression testing. |
| Operational Control | Reindexing, dashboards, failure queues. |
| Human Governance | Review AI enrichment suggestions before metadata updates. |

## 6. Release Gate Risks

Before MVP release, these must be closed or explicitly accepted:

- R-001: JRN rules not finalized.
- R-003: Chunk quality not reviewed.
- R-010: Security filters not enforced.
- R-014: Reindex controls missing.
- R-015: Source citations missing.
- R-016: Ambiguity handling missing.

## 7. Related Jira Stories

Risks should be linked to relevant Jira stories. Examples:

| Risk | Jira Stories |
| --- | --- |
| R-001 | DC-AI-011, DC-AI-012, DC-AI-014 |
| R-003 | DC-AI-030 through DC-AI-039 |
| R-010 | DC-AI-090 through DC-AI-097 |
| R-014 | DC-AI-110 through DC-AI-118 |
| R-016 | DC-AI-071, DC-AI-072 |
