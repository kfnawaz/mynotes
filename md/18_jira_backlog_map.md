# 18. Jira Backlog Map

## 1. Purpose

This page maps the Data Compass AI Modernization Confluence architecture package to the Jira initiative, epics, stories, labels, components, and delivery workflow.

## 2. Jira Hierarchy

```
Initiative: Data Compass AI Modernization
   -> Epic
      -> Story
         -> Sub-task
```

## 3. Initiative

**Name:** Data Compass AI Modernization

**Description:**

Deliver an AI-ready Data Compass metadata platform using MongoDB as the authoritative metadata source, vector search for semantic retrieval, Neo4j for graph relationship traversal, GraphRAG for grounded question answering, and a Context Plane for enterprise context and agent orchestration.

## 4. Epics

| Epic | Name | Description | Confluence Pages |
| --- | --- | --- | --- |
| EPIC-1 | Architecture and Planning | Define architecture, scope, decisions, risks, and delivery structure. | 00, 01, 02, 03, 15, 16, 17 |
| EPIC-2 | Canonical Metadata Foundation | Define normalized model, identity, relationships, lifecycle, classification. | 04 |
| EPIC-3 | Metadata Ingestion Service | Read, validate, resolve, and prepare MongoDB metadata. | 05 |
| EPIC-4 | AI Document and Chunk Builder | Build source-backed AI-readable chunks. | 06 |
| EPIC-5 | Embedding and Vector Indexing | Generate embeddings and vector index chunks. | 07 |
| EPIC-6 | Graph Projection and Neo4j Builder | Project metadata into Neo4j graph. | 08 |
| EPIC-7 | Data Compass Search Service | Provide exact, semantic, graph, hybrid search APIs. | 09 |
| EPIC-8 | GraphRAG Orchestration | Combine retrieval, graph expansion, context, answers. | 10 |
| EPIC-9 | Context Plane and Agent Layer | Build acronyms, Q&A, bookmarks, feedback, agents. | 11 |
| EPIC-10 | Security and Access Control | Enforce retrieval-level security. | 12 |
| EPIC-11 | Evaluation and Quality | Build golden questions and quality metrics. | 13 |
| EPIC-12 | Operations and Monitoring | Build reindexing, dashboards, reconciliation, runbooks. | 14 |
| EPIC-13 | UI / User Experience Integration | Integrate search, lineage, citations, feedback in UI. | 19 |

## 5. Labels

Use these Jira labels:

- data-compass-ai
- metadata
- graphrag
- vector-search
- mongodb
- neo4j
- lineage
- context-plane
- security
- mvp
- canonical-model
- ai-chunks
- evaluation
- operations

## 6. Components

Use these Jira components:

- Canonical Model
- Ingestion
- Chunk Builder
- Vector Index
- Neo4j Graph
- Search Service
- GraphRAG
- Context Plane
- Agents
- Security
- Evaluation
- Operations
- UI

## 7. Workflow

Recommended Jira workflow:

1. Backlog
1. Ready for Refinement
1. Ready for Sprint
1. In Progress
1. Blocked
1. In Review
1. Ready for Demo
1. Done

## 8. Definition of Ready

A story is Ready when:

- Objective is clear.
- Requirements are documented.
- Dependencies are identified.
- Acceptance criteria are written.
- Related Confluence page is linked.
- Security impact is understood.
- Test/evidence expectation is defined.

## 9. Definition of Done

A story is Done when:

- Work is completed.
- Acceptance criteria are met.
- Unit/integration tests are complete where applicable.
- Evidence is attached to Jira.
- Related Confluence page is updated.
- Demo is completed where required.
- Operational notes are added if applicable.

## 10. First Sprint Recommendation

Pull these first:

| Story | Reason |
| --- | --- |
| DC-AI-001 | Create Confluence structure. |
| DC-AI-002 | Document current-state architecture. |
| DC-AI-003 | Document target-state architecture. |
| DC-AI-004 | Create ADR log. |
| DC-AI-005 | Define MVP scope and phasing. |
| DC-AI-010 | Confirm MVP entity scope. |
| DC-AI-011 | Define GUID and JRN identity rules. |
| DC-AI-014 | Define relationship resolution rules. |
| DC-AI-015 | Define lifecycle status handling. |

## 11. Dependency Rules

- Do not start vector indexing until chunk schema and metadata filters are approved.
- Do not start Neo4j loaders until graph schema and jrn identity rules are approved.
- Do not start agents until basic search APIs work.
- Do not start answer generation until source citation strategy is defined.
- Do not launch MVP until security negative tests pass.
