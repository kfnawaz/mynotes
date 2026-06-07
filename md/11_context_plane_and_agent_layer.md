# 11. Context Plane and Agent Layer

## 1. Purpose

The Context Plane and Agent Layer improves interpretation, retrieval planning, answer quality, and feedback loops for Data Compass AI search. It provides reusable enterprise context such as acronyms, aliases, historical validated Q&A, user bookmarks, human feedback, and agent orchestration.

## 2. Role in Architecture

```
User Question
   -> Context Plane
      -> Acronyms / Aliases
      -> Bookmarks / Historical Q&A
      -> Feedback Signals
      -> Supervisor Agent
      -> Structured Discovery Agent
      -> Metadata Enrichment Agent
   -> Search Service / GraphRAG
```

## 3. Design Principles

| Principle | Description |
| --- | --- |
| Context assists, source data governs | Context improves interpretation but does not override authoritative metadata. |
| Agents are orchestrators | Agents route and structure work; they should not bypass controlled search APIs. |
| Human approval for enrichment | AI suggestions should not automatically update authoritative metadata. |
| Feedback is auditable | User feedback should be captured with question, answer, sources, and outcome. |
| User context is scoped | Bookmarks and preferences must be user-scoped and entitlement-aware. |

## 4. Context Plane Components

| Component | Responsibility | MVP Priority |
| --- | --- | --- |
| Acronyms Loader | Load enterprise acronyms and expansions. | P1 |
| Business Alias Manager | Map alternate business/technical names. | P1 |
| Historical Q&A Store | Store validated previous questions and answers. | P1 |
| User Bookmarks | Store user-specific important assets. | P2 |
| Human Feedback Store | Capture answer/result feedback. | P1 |
| Self-Evaluation Store | Store automated answer quality scores. | P2 |
| Context MCP Registry | Expose context tools to agents. | P2 |
| Redis Cache | Cache session and short-lived context. | P1 |

## 5. Agents

| Agent | Responsibility | Notes |
| --- | --- | --- |
| Supervisor Agent | Routes user request to correct flow. | First agent to build. |
| Structured Discovery Agent | Converts vague questions into structured search plans. | Uses acronyms and aliases. |
| Metadata Enrichment Agent | Suggests improved descriptions, tags, mappings, relationships. | Requires human approval. |
| Lineage Analysis Agent | Specializes in impact and dependency questions. | Uses Graph Search API. |
| Governance Agent | Handles classification, ownership, lifecycle, policy questions. | Uses filters and governance metadata. |
| Answer Evaluation Agent | Checks groundedness, source use, and completeness. | Supports quality loops. |

## 6. Acronyms and Alias Handling

Example:

```json
{
  "term": "CCR",
  "expansions": ["Counterparty Credit Risk", "Customer Contact Record"],
  "domain": "Risk",
  "confidence": 0.8,
  "source": "Enterprise Acronym List",
  "status": "approved"
}
```

Rules:

- Acronym expansion should be domain-aware.
- If an acronym has multiple meanings, GraphRAG should ask clarification or search multiple expansions.
- Acronym source and status should be tracked.

## 7. Historical Q&A Store

Purpose:

- Reuse validated answers.
- Improve retrieval examples.
- Identify recurring questions.
- Support evaluation and regression testing.

Schema:

```json
{
  "question": "",
  "answer": "",
  "sourceJrns": [],
  "userFeedback": "helpful | not_helpful | incorrect_source",
  "validatedBy": "",
  "validatedAt": "",
  "status": "draft | validated | deprecated"
}
```

## 8. User Bookmarks

Bookmarks support personalized discovery.

Rules:

- Bookmark must be tied to user identity.
- Bookmark must reference source jrn.
- Bookmark cannot bypass entitlements.
- Bookmarks may influence ranking, but not source truth.

## 9. Feedback Capture

Feedback types:

- Helpful / not helpful.
- Incorrect source.
- Missing source.
- Wrong lineage.
- Wrong entity match.
- Free-text feedback.

Feedback should capture:

- user
- question
- answer
- retrieved sources
- selected feedback
- timestamp
- route used

## 10. Supervisor Agent

Responsibilities:

1. Receive user question and context.
1. Determine intent.
1. Select retrieval strategy.
1. Route to Search Service / GraphRAG.
1. Decide whether clarification is needed.
1. Log selected route.

## 11. Structured Discovery Agent

Input:

```
Find important datasets related to customer onboarding risk.
```

Output:

```json
{
  "searchTerms": ["customer onboarding risk"],
  "entityTypes": ["Dataset", "Distribution", "DataProduct"],
  "filters": {
    "lifecycleStatus": ["PUBLISHED", "APPROVED"]
  },
  "includeGraphExpansion": true
}
```

## 12. Metadata Enrichment Agent

Enrichment suggestions may include:

- missing descriptions
- better display names
- tags
- glossary mappings
- owner recommendations
- relationship suggestions
- classification review flags

Rules:

- Suggestions must include evidence.
- Suggestions must be reviewable.
- Suggestions must not directly update authoritative metadata without approval.
- Approved changes should go through governed Data Compass update workflow.

## 13. Context Plane Storage

| Data Type | Suggested Store | Notes |
| --- | --- | --- |
| Session cache | Redis | Short-lived context only. |
| Acronyms | MongoDB / managed config | Versioned and approved. |
| Aliases | MongoDB / managed config | May be domain-scoped. |
| Historical Q&A | MongoDB | Source-backed and validated. |
| Feedback | MongoDB | Auditable. |
| Bookmarks | MongoDB | User-scoped. |
| Agent logs | Observability store | Do not log sensitive payloads unnecessarily. |

## 14. Security Requirements

- Context cannot expand user access.
- Bookmarks must not reveal inaccessible assets.
- Historical Q&A must be re-filtered at retrieval time.
- Feedback logs should avoid sensitive data values.
- Agent prompts must not include unauthorized metadata.

## 15. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| CTX-001 | Build acronyms loader | P1 |
| CTX-002 | Build business alias manager | P1 |
| CTX-003 | Build historical Q&A store | P1 |
| CTX-004 | Build user bookmarks | P2 |
| CTX-005 | Build feedback capture | P1 |
| CTX-006 | Build Supervisor Agent | P1 |
| CTX-007 | Build Structured Discovery Agent | P1 |
| CTX-008 | Build Metadata Enrichment Agent | P2 |
| CTX-009 | Build Answer Evaluation Agent | P2 |

## 16. Acceptance Criteria

- Acronyms can be loaded and used to expand search queries.
- Historical Q&A can be stored with source references.
- User feedback is captured with question and sources.
- Supervisor Agent routes exact, semantic, lineage, and governance questions correctly.
- Metadata Enrichment Agent produces suggestions with evidence and no direct write to authoritative metadata.
- Context Plane does not bypass access control.

## 17. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-080 | Build Acronyms Loader |
| DC-AI-081 | Build Business Alias Manager |
| DC-AI-082 | Build Historical Q&A Store |
| DC-AI-083 | Build User Bookmarks |
| DC-AI-084 | Build Feedback Capture |
| DC-AI-085 | Build Supervisor Agent |
| DC-AI-086 | Build Structured Discovery Agent |
| DC-AI-087 | Build Metadata Enrichment Agent |
| DC-AI-088 | Build Answer Evaluation Agent |

## 18. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-CTX-001 | What is the authoritative source for enterprise acronyms? | Loader design. |
| OQ-CTX-002 | Should historical Q&A be globally shared or domain-scoped? | Security and relevance. |
| OQ-CTX-003 | Who approves metadata enrichment suggestions? | Governance workflow. |
| OQ-CTX-004 | Should bookmarks influence ranking in MVP? | UX and ranking. |
