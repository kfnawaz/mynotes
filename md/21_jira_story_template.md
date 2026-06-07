# 21. Jira Story Template

## Purpose

Use this template for all Jira stories in the Data Compass AI Modernization initiative. The goal is to ensure every story has enough context, requirements, pitfalls, dependencies, and acceptance criteria before sprint commitment.

## Story Template

```
h2. Objective

Describe what this story delivers in one or two sentences.

h2. Background

Explain why this story is needed and how it fits into Data Compass AI Modernization.

h2. Scope

h3. In Scope

* ...

h3. Out of Scope

* ...

h2. Requirements

* Requirement 1
* Requirement 2
* Requirement 3

h2. Inputs

|| Input || Source || Notes ||
|  |  |  |

h2. Outputs

|| Output || Consumer || Notes ||
|  |  |  |

h2. Dependencies

* Dependency 1
* Dependency 2

h2. Pitfalls / Risks

* Risk 1
* Risk 2

h2. Security Considerations

* Does this story touch metadata access?
* Does this story expose sensitive metadata?
* Are lifecycle/classification filters required?
* Are audit logs required?

h2. Acceptance Criteria

* [ ] Criterion 1
* [ ] Criterion 2
* [ ] Criterion 3

h2. Test / Evidence

* Unit test:
* Integration test:
* Demo evidence:
* Screenshot/API response/log link:

h2. Related Confluence

* Link to relevant Confluence page.

h2. Related ADRs

* ADR-###

h2. Notes

Additional implementation or review notes.
```

## Example Story: Build Distribution Summary Chunk Template

```
h2. Objective

Build the chunk template that converts a Distribution source record into an AI-readable asset_summary chunk.

h2. Background

Distribution is the primary physical asset in the Data Compass model. It represents physical data assets such as Databricks or Snowflake tables/views and is a key search target for AI-powered metadata discovery.

h2. Scope

h3. In Scope

* Generate asset_summary text for Distribution.
* Include source guid, jrn, entity type, lifecycle status, and version.
* Include physical specification fields.
* Include owners and node context where available.
* Include related Dataset jrn when resolved.
* Generate deterministic chunk_id.

h3. Out of Scope

* Embedding generation.
* Vector upsert.
* Neo4j graph loading.
* UI display.

h2. Requirements

* Template must produce readable text, not raw JSON.
* Template must include source traceability fields.
* Template must include enough context for semantic search.
* Template must exclude secrets and sensitive connection values.

h2. Inputs

|| Input || Source || Notes ||
| Distribution record | MongoDB | Required. |
| Resolved Dataset | Relationship Resolver | Optional if link exists. |
| Owners | Distribution owners[] | Optional but should be included when present. |
| Node | Node Resolver | Optional but important for filters. |

h2. Outputs

|| Output || Consumer || Notes ||
| asset_summary chunk | Embedding Service | Persisted before embedding. |

h2. Dependencies

* DC-AI-030 Define AI Chunk Schema
* DC-AI-011 Define GUID and JRN Identity Rules
* DC-AI-014 Define Relationship Resolution Rules

h2. Pitfalls / Risks

* Raw JSON text reduces search quality.
* Missing jrn breaks traceability.
* Large attribute lists may make the chunk too long.
* Sensitive connection details must not appear in chunk text.

h2. Acceptance Criteria

* [ ] Given a valid Distribution, one asset_summary chunk is created.
* [ ] Chunk includes source guid and source jrn.
* [ ] Chunk includes entity type Distribution and lifecycle status.
* [ ] Chunk includes physical specification summary.
* [ ] Chunk includes related Dataset when available.
* [ ] Chunk ID is deterministic.
* [ ] Chunk text is readable and does not expose secrets.

h2. Test / Evidence

* Sample input Distribution record.
* Generated chunk output.
* Unit test for deterministic chunk ID.
* Review sample attached to Jira.
```

## Definition of Ready

A story is Ready when:

- Objective is clear.
- Requirements are documented.
- Dependencies are identified.
- Acceptance criteria are written.
- Related Confluence page is linked.
- Security impact is understood.
- Test/evidence expectation is defined.

## Definition of Done

A story is Done when:

- Work is completed.
- Acceptance criteria are met.
- Unit/integration tests are complete where applicable.
- Evidence is attached to Jira.
- Related Confluence page is updated.
- Demo is completed where required.
- Operational notes are added if applicable.
