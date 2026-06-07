# 13. Evaluation and Success Metrics

## 1. Purpose

This page defines how Data Compass AI Modernization will be evaluated for retrieval quality, graph correctness, answer groundedness, access-control safety, operational reliability, and user value.

## 2. Evaluation Principles

| Principle | Description |
| --- | --- |
| Test retrieval separately from generation | First prove search works, then evaluate answers. |
| Use golden questions | Maintain a reviewed set of representative questions and expected results. |
| Measure source grounding | Answers must be supported by source metadata. |
| Test negative cases | Missing data and restricted data must be tested. |
| Track regressions | Every release should be measured against prior results. |

## 3. Evaluation Categories

| Category | What It Measures |
| --- | --- |
| Exact Search | Ability to find known assets by identifiers and fields. |
| Semantic Search | Ability to retrieve relevant chunks for business-language questions. |
| Graph Search | Correctness of upstream/downstream and dependency traversal. |
| GraphRAG | Correct retrieval strategy, context assembly, and answer generation. |
| Security | Unauthorized retrieval prevention. |
| Answer Quality | Groundedness, clarity, completeness, and citations. |
| Operations | Indexing, reindexing, sync, and reliability. |
| User Experience | Helpfulness and ease of interpretation. |

## 4. MVP Success Targets

| Metric | MVP Target |
| --- | --- |
| Exact lookup accuracy | 95%+ |
| Top-5 semantic retrieval accuracy | 80%+ |
| Top-3 semantic retrieval accuracy | 70%+ |
| Lineage traversal correctness | 90%+ |
| Source citation coverage | 95%+ |
| Unauthorized retrieval prevention | 100% |
| Unsupported answer claims | Near zero for golden set. |
| Vector upsert success rate | 95%+ for valid chunks. |
| Graph load reconciliation | 95%+ for in-scope records. |
| Reindex success rate | 95%+ |

## 5. Golden Question Set

Golden questions should be stored with:

- question ID
- question text
- intent category
- expected source jrns
- expected answer outline
- required filters
- expected no-answer behavior if applicable
- owner/reviewer
- status

### Golden Question Categories

| Category | Examples |
| --- | --- |
| Exact Lookup | Find Distribution by JRN; find Dataset by name; find Report by report ID. |
| Semantic Discovery | Find datasets related to customer risk; find attributes representing customer identifiers. |
| Lineage | What is upstream of asset X? What is downstream of asset Y? |
| Impact Analysis | Which reports depend on Dataset X? |
| Governance | Which assets contain PII? Which are highly confidential? |
| Ambiguity | Show customer table when multiple customer tables exist. |
| Missing Data | Ask for owner when owner metadata is absent. |
| Security | Ask for restricted asset without access. |

## 6. Retrieval Evaluation

### Exact Search Metrics

- exact match found
- correct entity type
- correct source jrn
- correct lifecycle filtering
- correct owner/node filtering

### Semantic Search Metrics

- Top-1 accuracy
- Top-3 accuracy
- Top-5 accuracy
- mean reciprocal rank
- irrelevant result rate
- duplicate result rate
- missing source reference rate

## 7. Graph Evaluation

Validate:

- upstream paths
- downstream paths
- depth limits
- dependency relationships
- relationship direction
- orphan/missing relationships
- restricted path filtering

## 8. Answer Quality Evaluation

| Quality Dimension | Expected Behavior |
| --- | --- |
| Groundedness | Claims are supported by retrieved source records. |
| Citation coverage | Answer includes source references. |
| Completeness | Answer covers the requested scope. |
| Clarity | Answer is understandable to intended user. |
| Honesty | Missing metadata is stated clearly. |
| Safety | Restricted metadata is not revealed. |

## 9. Hallucination Tests

Test prompts where the correct answer is:

- metadata is missing
- asset does not exist
- relationship does not exist
- user lacks access
- multiple assets match and clarification is needed

Expected behavior: system should not invent an answer.

## 10. Evaluation Report Format

| Run Date | Build Version | Test Set | Pass Rate | Failures | Owner | Notes |
| --- | --- | --- | --- | --- | --- | --- |
|  |  | Exact Search |  |  |  |  |
|  |  | Semantic Search |  |  |  |  |
|  |  | Graph Search |  |  |  |  |
|  |  | GraphRAG Answer Quality |  |  |  |  |
|  |  | Security Negative Tests |  |  |  |  |

## 11. Build Requirements

| ID | Requirement | Priority |
| --- | --- | --- |
| EVAL-001 | Create golden question set | P0 |
| EVAL-002 | Define expected answers and source jrns | P0 |
| EVAL-003 | Build retrieval quality evaluation | P1 |
| EVAL-004 | Build lineage correctness evaluation | P1 |
| EVAL-005 | Build answer quality evaluation | P1 |
| EVAL-006 | Build hallucination test cases | P1 |
| EVAL-007 | Build regression report | P1 |
| EVAL-008 | Create human review workflow | P1 |

## 12. Acceptance Criteria

- Golden question set exists with expected results.
- Evaluation can measure exact, semantic, graph, and answer quality separately.
- Security negative tests are included.
- Failed tests produce actionable records.
- Regression results are stored and reviewed before release.

## 13. Related Jira Stories

| Jira | Summary |
| --- | --- |
| DC-AI-100 | Create Golden Question Set |
| DC-AI-101 | Define Expected Answers |
| DC-AI-102 | Build Retrieval Quality Evaluation |
| DC-AI-103 | Build Lineage Correctness Evaluation |
| DC-AI-104 | Build Answer Quality Evaluation |
| DC-AI-105 | Build Hallucination Test Cases |
| DC-AI-106 | Build Regression Test Report |
| DC-AI-107 | Create Human Review Workflow |

## 14. Open Questions

| ID | Question | Impact |
| --- | --- | --- |
| OQ-EVAL-001 | Who owns golden question approval? | Governance of evaluation set. |
| OQ-EVAL-002 | What pass rate is required for production launch? | Release gate. |
| OQ-EVAL-003 | Should evaluations run automatically in CI/CD? | Delivery process. |
