# 10. GraphRAG Orchestration

## 1. Purpose

GraphRAG Orchestration combines exact search, semantic vector retrieval, graph traversal, source hydration, context assembly, and answer generation into a grounded question-answering flow.

The goal is to answer metadata questions using both meaning and relationships. Vector search finds relevant concepts; graph traversal explains lineage, impact, dependencies, and connected context.

## 2. Role in Architecture

```
User Question
   -> Intent Classifier
   -> Entity Resolver
   -> Retrieval Planner
   -> Search Service
   -> Graph Expansion
   -> Source Hydration
   -> Context Assembler
   -> Answer Composer
   -> Answer Evaluation
```

## 3. Design Principles

| Principle | Description |
| --- | --- |
| Retrieval before generation | The LLM should answer from retrieved metadata, not memory. |
| Graph where relationships matter | Use Neo4j for lineage, impact, and dependency questions. |
| Semantic where meaning matters | Use vector search for discovery and business-language questions. |
| Exact where identity matters | Use exact lookup for jrn, guid, names, schemas, report IDs. |
| Source-backed answers | Every answer should include source references where possible. |
| No hallucination | If metadata is missing, say so clearly. |
| Clarify ambiguity | Ask user to choose when multiple assets match. |

## 4. Supported Question Types

| Question Type | Example | Retrieval Strategy |
| --- | --- | --- |
| Exact Lookup | Show me Distribution X. | Exact-first. |
| Semantic Discovery | Find assets related to customer onboarding risk. | Semantic-first. |
| Lineage | What is upstream of table X? | Graph-first. |
| Impact Analysis | What reports depend on Dataset X? | Exact + graph. |
| Governance | Which assets contain PII? | Filtered exact/semantic. |
| Service Exposure | Which services expose Distribution X? | Exact + graph. |
| Ambiguous | Show customer table. | Entity resolver + clarification. |
| Missing Data | Who owns an asset with no owner metadata? | Search + missing-data response. |

## 5. Intent Classifier

Classifier outputs:

```json
{
  "intent": "exact_lookup | semantic_discovery | lineage | impact_analysis | governance | enrichment | ambiguous",
  "entitiesMentioned": [],
  "candidateFilters": {},
  "requiresGraph": true,
  "requiresSemantic": false,
  "requiresExact": true,
  "confidence": 0.87
}
```

## 6. Entity Resolver

Responsibilities:

- Resolve mentioned names to canonical entities and jrns.
- Search exact identifiers first.
- Search aliases and acronyms through Context Plane.
- Handle multiple matches.
- Ask clarifying question when needed.

Resolution output:

```json
{
  "resolved": true,
  "selectedEntity": {
    "jrn": "",
    "guid": "",
    "entityType": "Distribution",
    "name": "customer_transactions"
  },
  "candidates": [],
  "needsClarification": false
}
```

## 7. Retrieval Strategies

### Exact-First Flow

```
1. Exact search by name/guid/jrn/schema/report ID.
2. Resolve one or more candidates.
3. Hydrate source record.
4. Retrieve related chunks.
5. Compose answer.
```

### Semantic-First Flow

```
1. Expand query with acronyms/aliases.
2. Run vector search with filters.
3. Deduplicate by source jrn.
4. Optionally expand top results through graph.
5. Hydrate selected sources.
6. Compose answer.
```

### Graph-First Flow

```
1. Resolve starting asset.
2. Run graph traversal with direction and depth.
3. Apply security filters.
4. Hydrate nodes and relationships.
5. Retrieve supporting chunks.
6. Compose lineage/impact answer.
```

### Hybrid Flow

```
1. Run exact search for explicit entities.
2. Run semantic search for meaning.
3. Use graph expansion for relationships.
4. Rerank and deduplicate.
5. Hydrate sources.
6. Compose source-backed answer.
```

## 8. Context Assembler

The Context Assembler creates bounded context for the LLM.

Inputs:

- search results
- source records
- graph paths
- chunks
- user context
- acronyms and aliases
- prior validated Q&A where relevant

Rules:

- Deduplicate by source jrn.
- Prefer authoritative source records over derived chunks.
- Respect token limits.
- Preserve source references.
- Exclude unauthorized context.
- Include enough graph path detail for lineage answers.

## 9. Answer Composer

Answer must:

- Directly answer the user question.
- Use only provided context.
- Include source references.
- State missing metadata clearly.
- Show lineage paths when requested.
- Avoid claims not supported by retrieved records.
- Ask clarification when ambiguity remains.

## 10. Output Types

| Output Type | Use |
| --- | --- |
| Summary Answer | General metadata discovery. |
| Asset List | Search results and recommendations. |
| Lineage Path | Upstream/downstream questions. |
| Impact Analysis | Downstream reports/processes/products. |
| Governance Summary | Classification/ownership/retention. |
| Clarification Prompt | Multiple candidate entities. |
| No-Answer Response | Missing or unauthorized metadata. |

## 11. Example: Downstream Reports

Question:

```
What downstream reports depend on customer_transactions?
```

Flow:

1. Resolve customer_transactions to Distribution jrn.
1. Traverse DOWNSTREAM_OF and DEPENDS_ON relationships.
1. Filter results by user access.
1. Hydrate Report records from MongoDB.
1. Retrieve supporting chunks.
1. Return report list and lineage path.

## 12. Failure and Guardrail Behavior

| Scenario | Expected Behavior |
| --- | --- |
| No matching entity | State no matching asset was found and suggest search refinements. |
| Multiple matching entities | Ask user to choose from candidates. |
| No lineage found | State no lineage is currently available for the asset. |
| Restricted result | Exclude result; optionally state some results may be hidden by access rules. |
| Source hydration failed | Return warning and avoid unsupported details. |
| Low retrieval confidence | Ask clarification or present candidates instead of over-answering. |

## 13. Evaluation Metrics

| Metric | Target |
| --- | --- |
| Intent classification accuracy | 90%+ MVP test set. |
| Entity resolution accuracy | 90%+ for exact names. |
| Source-backed answer coverage | 95%+. |
| Unsupported claims | Near zero. |
| Clarification correctness | Ambiguous questions produce candidate choices. |
| Lineage path correctness | 90%+ against golden paths. |

## 14. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| RAG-001 | Build question intent classifier | P0 |
| RAG-002 | Build entity resolver | P0 |
| RAG-003 | Build retrieval planner | P0 |
| RAG-004 | Build graph expansion service | P1 |
| RAG-005 | Build context assembler | P1 |
| RAG-006 | Build answer composer | P1 |
| RAG-007 | Build source citation handling | P1 |
| RAG-008 | Build ambiguity handling flow | P1 |
| RAG-009 | Build missing metadata response behavior | P1 |

## 15. Acceptance Criteria

- GraphRAG selects the correct retrieval strategy for golden questions.
- Exact asset questions resolve to the correct canonical jrn.
- Lineage questions use graph traversal, not only vector search.
- Semantic discovery questions use vector search and optional graph expansion.
- Answers include source references.
- Missing or ambiguous metadata is handled honestly.
- Restricted context is not passed to the LLM.

## 16. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-070 | Build Question Intent Classifier |
| DC-AI-071 | Build Entity Resolver |
| DC-AI-072 | Build Ambiguity Handling Flow |
| DC-AI-073 | Build Graph Expansion Service |
| DC-AI-074 | Build Context Assembler |
| DC-AI-075 | Build Answer Composer |
| DC-AI-076 | Build Missing Metadata Response Behavior |
| DC-AI-077 | Build Lineage Answer Template |

## 17. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-RAG-001 | Which LLM endpoint/model is approved for answer generation? | Implementation. |
| OQ-RAG-002 | What citation format should the UI display? | UX and API design. |
| OQ-RAG-003 | Should the orchestrator return answers or only structured context in MVP? | Architecture split. |
| OQ-RAG-004 | What confidence threshold triggers clarification? | User experience. |
