For your DataCompass Crawlers modernization project, I would implement a stricter version of Spec-Driven Development called Contract-First, Specification-Governed Development.

You already have a substantial foundation: architecture documents, 12 proposed ADRs, 19 implementation work orders (P00–P18), a 20-row Atlan parity matrix, and agent instructions.

What is missing is the enforcement mechanism that ensures the next AI agent:
- Builds only what the specifications authorize.
- Reproduces required behavior from the existing Python and TypeScript implementations.
- Uses Java for the main implementation and Python only for SQLGlot.
- Does not invent answers to unresolved architecture questions.
- Keeps the specifications updated as development proceeds.
- Proves functionality through independent automated tests.
My recommendation is to preserve your existing documentation and prompt pack, but put five enforceable gates around it.
1. Your current position
DataCompass Crawlers — SDD readiness
Design baseline only

Architecture documentation	Drafted
Implementation prompt pack	P00–P18
Architecture decisions	12 proposed
Open questions	19 unresolved
Machine-verifiable contracts	Not demonstrated
Automated parity tests	Not demonstrated
Specification enforcement in CI	Not demonstrated
Git baseline	Uncommitted

Assessment based on the agent's report in your screenshots, not an independent inspection of the repositories.
Your documentation is a promising design package, but it is not yet an enforceable specification system. Markdown links passing validation does not prove the technical specifications are complete, consistent, or implementable.
2. The five enforcement gates

Gate 1 — Specification baseline approval

No implementation of a component until its applicable requirements, architecture decisions, dependencies, and contracts are reviewed and approved.




Gate 2 — Legacy behavior verification

Every required parity capability has a traceable source behavior, expected result, and acceptance test before migration work begins.




Gate 3 — Controlled AI implementation

Claude/Copilot builds one approved work order at a time, using only permitted technology and changing no unrelated functionality.




Gate 4 — Automated compliance

GitHub Actions validates requirements, traceability, tests, allowed technologies, architecture contracts, and documentation changes.




Gate 5 — Independent conformance review

A separate AI reviewer checks whether the implementation satisfies the approved specifications, with objective test evidence and human approval for material decisions.



3. Extend the existing repository, don't recreate the documentation
You already have docs/00–15, docs/adr/, and docs/prompts/. Keep them.

Add the following small set of specification-control artifacts.
datacompass-crawlers/
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── docs/
│   ├── 00-index.md
│   ├── 01-... through 15-...
│   ├── adr/                  # Existing
│   ├── prompts/              # Existing
│   ├── governance/
│   │   ├── constitution.md
│   │   ├── definition-of-done.md
│   │   └── approval-policy.md
│   ├── requirements/
│   │   └── requirements.yaml
│   ├── contracts/
│   │   ├── canonical-entity.schema.json
│   │   ├── lineage-event.schema.json
│   │   ├── crawler-run.schema.json
│   │   ├── control-plane.openapi.yaml
│   │   └── sqlglot-boundary.md
│   ├── parity/
│   │   ├── atlan-sync-matrix.yaml
│   │   └── fixtures/
│   └── traceability.yaml
├── tests/
│   ├── contracts/
│   ├── parity/
│   └── integration/
├── scripts/
│   └── validate-specs.*
└── .github/
    ├── workflows/
    │   └── sdd-gate.yml
    └── CODEOWNERS



The distinction is important:
Existing artifact	Additional enforcement artifact
Architecture documents	Explicitly approved architecture baseline
ADRs	Machine-checkable decision status and dependencies
20-row parity matrix	Executable tests and fixtures for each required behavior
Canonical data contract document	Versioned JSON schemas or equivalent contracts
Control-plane API design	OpenAPI specification and contract tests
P00–P18 prompts	Work orders linked to requirement IDs and verification evidence
Coding standards	CI policies, linters, build rules, and security checks

Don't make two independently maintained sources of truth. The machine-readable files should reference the existing documents, or be generated from a single canonical source.
4. Resolve the most important ambiguities before coding
Your screenshots already reveal issues that could cause different agents to build incompatible systems.

I would classify the open questions as follows:
Decision	Required action	Enforcement
Q-04: Atlan-authored protection flags and business terms	Assign migration owner and define preservation rules	Block destructive cutover
Q-05: Unity Catalog write-back selection	Establish exact criteria, ownership, and trigger behavior	Block write-back implementation
Q-06: Lineage relationship ownership	Agree responsibility with lineage-app team	Block dependent lineage integration
Q-10: Databricks JDBC artifact	Confirm the approved dependency	Block JDBC-specific implementation
Q-11: Python sidecar on GAIA	Verify deployment feasibility	Block sidecar integration
Q-17: SEAL identifier	Get approved configuration	Block affected environment deployment
Q-19: WMA/metadata-app scope	Agree service boundaries	Block duplicate API implementation

The 12 proposed ADRs also need review, but I wouldn't require all 12 to be approved before any work starts.

Instead, each work order must declare its dependencies. Only work affected by an unresolved decision should be blocked. Read-only discovery, contract drafting, independent test preparation, and other nondependent tasks can proceed.

For example, the P00 onboarding/audit work can remain read-only. P01 scaffolding should proceed only after the decisions relevant to the foundational architecture are approved.

Important: A coding agent must never convert Proposed to Approved on its own. Human approval is required, with decision owner, date, and evidence.
5. Turn the parity matrix into executable specifications
This is probably the most valuable change for your project.

Your new Java implementation should not be validated only against written descriptions. It should be compared against the required behavior of the original Python and TypeScript systems.

Use this pattern:
LEGACY IMPLEMENTATION

Python / TypeScript

TEST EVIDENCE

Golden fixtures




Approved behavior contracts
Input, expected output, invariants, error handling, performance expectations

New Java implementation + SQLGlot sidecar


 Automated contract and parity comparison

Matching required behavior, not necessarily identical code or internal architecture

For each of the 20 parity rows, establish the classification:
- Parity: Must preserve observed and approved existing behavior.
- Enhancement: New behavior, intentionally absent in the legacy implementation.
- Correction: Existing behavior is incorrect and the new implementation must follow an approved corrected contract.
- Migration: Data, metadata, or ownership must be transferred or preserved.
For instance, your screenshot says owner mapping for tables and columns is not actually implemented in the sync app. That should be classified as an enhancement, not falsely marked as existing parity.

Similarly, PASS_THROUGH, lineage events, provenance, and lifecycle states need explicit compatibility decisions. The new canonical model must not silently introduce values that existing downstream consumers cannot understand.

The golden fixtures should cover happy paths, failures, nulls, changes, deletions, lineage cases, permission boundaries, and idempotency. Pin the reference repositories to specific commit SHAs and sanitize any captured metadata.

Do not blindly copy legacy output into new requirements. Where legacy behavior is wrong, have the domain owner approve the intended behavior and corresponding tests.
6. Establish a constitution that every coding agent must follow
This is where you can enforce your Java-first migration objective.

Below is the instruction I would give the agent responsible for strengthening the existing repository.
Task: Establish Enforceable Spec-Driven Development for DataCompass Crawlers
You are the Specification Governance Architect for the datacompass-crawlers repository.
Objective
Convert the existing design documentation and implementation prompt pack into an enforceable Spec-Driven Development system that allows a separate AI agent to rebuild the application from approved specifications, without relying on undocumented knowledge of the legacy implementation.

This is a governance and specification-hardening task. Do not implement application functionality.
Existing Artifacts
Preserve and inspect:
- README.md
- AGENTS.md
- CLAUDE.md
- docs/00–15
- docs/adr/
- docs/prompts/, including P00–P18 and P99
- The 20-row Atlan-sync parity matrix
- The existing open questions and proposed architecture decisions
Do not replace existing specifications or silently change their meaning.
Mandatory Architecture Rules
1. The primary application must be implemented in Java.
2. Python is permitted only within a clearly isolated SQLGlot integration boundary.
3. Python must not implement the main crawler orchestration, connector SPI, control plane, canonical persistence, or run-state management.
4. The Python sidecar deployment is conditional on approval of Q-11.
5. Reuse the approved Java application and platform architecture.
6. Preserve required existing behavior from the Python and TypeScript implementations.
7. Document deviations from legacy behavior explicitly as approved enhancements or corrections.
8. Do not invent architecture decisions, business rules, or answers to open questions.
9. Do not introduce undocumented database fields, API responses, lifecycle states, or placeholder metadata.
10. Preserve ownership of metadata-app read APIs unless a boundary change is approved.
Required Deliverables
A. Specification Governance
Create a project constitution, specification lifecycle, approval policy, and Definition of Done.

Define statuses: Draft, In Review, Approved, Implemented, Verified, and Superseded.

A specification must not become Approved merely because an AI agent has completed it.
B. Traceability
Establish machine-readable mappings connecting:

Requirement → Source Evidence → Approved Decision → Design Contract → Work Order → Implementation → Automated Tests.

Every parity row must have a unique identifier, classification, acceptance criteria, and verification strategy.
C. Machine-Readable Contracts
Identify and formalize the canonical entity schema, lineage events, crawler run lifecycle, connector SPI contracts, control-plane APIs, and Java/Python SQLGlot boundary.

Contracts must specify validation, failure behavior, compatibility, and versioning rules.
D. Agent Instructions
Update AGENTS.md and CLAUDE.md to require reading approved specifications and relevant ADRs before implementation.

The agent must identify unresolved dependencies and stop affected work without inventing an answer.
E. Automated Validation
Create an SDD validation mechanism suitable for GitHub Actions that checks:
- Requirement IDs and traceability references
- Approval dependencies for affected work orders
- Spec changes accompanying behavior changes
- Test coverage references
- Schema and OpenAPI validation
- Architecture constraints
- Documentation integrity
Do not claim that a text-based checklist proves functional parity.
F. Independent Review
Define a reviewer workflow that evaluates specifications and implementation independently.

The implementing agent must not self-approve architecture decisions, requirements exceptions, or successful parity.
Completion Requirements
Provide:
1. Files created or modified.
2. Specification gaps discovered.
3. Unresolved decisions and their affected work orders.
4. Automated checks implemented and actual results.
5. Remaining manual approval requirements.
6. A recommended order for beginning P00 and P01.
Do not begin application coding, approve ADRs, or modify unrelated repositories.



This prompt gives the first agent a clearly bounded job: harden the specification system before handing the project to the implementation agent.
7. Make GitHub the enforcement authority
This is critical because CLAUDE.md and AGENTS.md influence behavior but cannot reliably guarantee compliance. Claude's own guidance recommends using hooks or permission controls for deterministic restrictions rather than relying solely on written instructions. 

Claude by Anthropic

I would configure a protected main branch, required pull requests, CODEOWNERS review, and mandatory GitHub Actions status checks.

GitHub supports these controls through branch protection and repository rulesets. 

GitHub Docs+1

Set up the following checks:
Automated check	Condition to pass
Specification integrity	Valid IDs, references, links, and required fields
Requirement traceability	Each changed behavior maps to a known requirement
ADR dependencies	All decisions required by the work order are approved
Technology restrictions	No unauthorized Python application code
Contract validation	JSON schemas, Java interfaces, and OpenAPI compatibility checks
Parity verification	Required fixtures pass against the Java implementation
Quality verification	Java build, unit, integration, security, and static analysis pass
Spec synchronization	Behavior changes have corresponding specification and test changes

For code changes, have CI require an explicit mapping to approved requirement IDs. Implement a narrowly scoped exception mechanism for nonbehavioral refactoring, with human review.

For architecture restrictions, don't rely on a filename check alone. Add Java dependency rules (for example, ArchUnit), module boundaries, and package dependency tests.

The agents can run local hooks and pre-commit checks for faster feedback, but the protected-branch checks should remain authoritative.
8. Define a strict work-order lifecycle for P00–P18
Your existing prompts can become the official execution units.

Each work order should move through these states:
01

Draft


02

Reviewed


03

Approved


04

Implemented


05

Tested


06

Verified



Approval and verification are distinct from implementation. Failed verification returns the work order to implementation or specification review.
For every P01–P18 implementation run, enforce this sequence:
1. Agent loads the approved specifications and dependencies for the selected work order.
2. Agent checks that prerequisite decisions and contracts are approved.
3. Agent creates or finalizes acceptance tests against approved behavior.
4. Agent implements only the authorized scope.
5. CI runs contract, regression, parity, and other applicable tests.
6. Independent reviewer validates traceability, deviations, and implementation evidence.
7. Human reviewer merges the change after required gates pass.
Use P99 as an independent review prompt, but strengthen it so it produces a structured conformance report, not simply a narrative code review.

Also, use a different agent session for review, ideally without the implementation agent's prior conversation. That reduces the chance that the reviewer accepts assumptions made during coding.
9. Consider Spec Kit, but don't allow it to overwrite your design
GitHub's Spec Kit can help operationalize this.

Its workflow includes constitution, specification, clarification, planning, checklists, tasks, consistency analysis, implementation, and convergence review. 

Spec Kit Documentation

However, since your repository already has detailed design documentation and P00–P18 work orders, I would adapt Spec Kit to the existing structure, not regenerate the project from a blank template.

Use its requirements-quality analysis and consistency checks, while keeping the existing canonical documentation and decision records.

One thing especially worth defining in your project constitution is specification persistence: whether feature specifications remain authoritative after implementation. For your reconstruction goal, I strongly recommend that approved product and technical contracts remain permanent and versioned, not disposable planning artifacts.
10. Establish measurable reconstruction readiness
I would track the following metrics on every meaningful release.
Proposed release targets, not measured project resultsRequirement coverage

Target 100%



Approved requirements with mapped automated verification
Parity coverage

Target 100%



Required parity rows verified by reference fixtures
Contract coverage

Target 100%



Required external and inter-module contracts validated
Traceability coverage

Target 100%



Implemented work orders linked to specs and tests

Alongside these, track unresolved high-risk decisions, known specification ambiguities, and the results of independent clean-room rebuild tests.

A 100% score on the traceability metrics is not proof that the entire system can be reconstructed. The final evidence comes from an independent implementation built using only approved specifications and its successful execution against the same acceptance suite.
11. What I would do immediately
Given the state of the project in your screenshots, I recommend this order:
Priority	Action	Expected outcome
1	Commit the current documentation as a clearly labeled draft baseline	Reproducible starting point
2	Run a specification completeness and contradiction audit	Missing and inconsistent requirements identified
3	Review relevant ADRs and assign owners to blocking questions	Controlled approval path
4	Convert the 20 parity rows into testable contracts	Migration behavior becomes measurable
5	Add governance files and GitHub Actions controls	Agents cannot merge nonconforming changes
6	Execute P00 read-only and review its findings	Verify foundational assumptions
7	Begin approved P01 scaffolding	Controlled implementation starts
8	Execute subsequent prompts with independent verification after each	Continuous spec/code consistency


The most important change I recommend
Right now your workflow is approximately:
Legacy repositories  AI-generated documentation  AI-generated implementation

Change it to:
Legacy repositories + verified observations

Reviewed specifications + approved contracts

Acceptance tests + traceability

Claude/Copilot implementation

Automated compliance + independent verification

Versioned, reproducible release

For DataCompass specifically, I would make the approved canonical data model, connector SPI, crawler run lifecycle, and SQLGlot interoperability contract the central technical specifications. Those are the boundaries most likely to cause expensive incompatibilities when moving from Python/TypeScript to Java.

The critical next action is not P01 coding. It is having the specification agent convert the existing documents, decisions, parity matrix, and prompts into a governed, testable baseline.

Once that is established, Claude Code or Copilot becomes an interchangeable implementation engine, while the specifications, contracts, and acceptance tests remain the long-term assets of your project.


# DataCompass Crawlers — Spec-Driven Development (SDD) Strategy

## 1. Objective

Establish a strict **Contract-First, Specification-Governed Development** process for the DataCompass Crawlers modernization project.

The goal is to ensure that:

1. The entire application can be rebuilt from approved specifications.
2. The primary implementation is Java.
3. Python is used only for SQLGlot integration.
4. Required existing functionality from Python and TypeScript is preserved.
5. AI agents do not invent business rules or architectural decisions.
6. Specifications and implementation remain synchronized.
7. Every feature is validated against automated acceptance tests.
8. Different AI agents or development teams can independently implement the application.

### Guiding Principle

> Specifications are the authoritative definition of the product.
> Source code is an implementation of those specifications.
> Automated tests verify whether the implementation conforms to the specifications.

---

## 2. Current Project Assessment

The existing `datacompass-crawlers` repository already contains substantial design documentation and an implementation prompt pack.

| Area | Current State |
|---|---|
| Architecture documentation | Drafted |
| Implementation prompts | P00–P18 |
| Architecture Decision Records | 12 proposed ADRs |
| Open questions | 19 unresolved |
| Atlan-sync parity matrix | 20 rows |
| Agent instructions | AGENTS.md and CLAUDE.md |
| Machine-verifiable contracts | Not demonstrated |
| Automated parity tests | Not demonstrated |
| Specification enforcement in CI | Not demonstrated |
| Documentation baseline | Not committed |

**Assessment:** The repository has a promising design foundation but does not yet have an enforceable specification system.

Markdown link validation alone does not establish that the specifications are complete, consistent, or implementable.

---

## 3. Five Mandatory SDD Enforcement Gates

### Gate 1 — Specification Baseline Approval

No implementation of a component begins until its applicable specifications are reviewed and approved.

Requirements:

- Functional requirements are documented.
- Architecture decisions are identified.
- Technical contracts are defined.
- Dependencies are identified.
- Acceptance criteria are established.
- Open questions affecting implementation are resolved.

Human approval is mandatory for architectural and business decisions.

### Gate 2 — Legacy Behavior Verification

Every required parity capability must have:

- Reference to existing behavior.
- Source repository and commit reference.
- Defined inputs and expected outputs.
- Expected error handling.
- Edge cases.
- Automated acceptance tests.

The new Java implementation must preserve approved legacy behavior.

### Gate 3 — Controlled AI Implementation

Claude Code or GitHub Copilot must:

1. Read the approved specifications.
2. Identify the relevant requirement IDs.
3. Verify prerequisite ADR approvals.
4. Implement only the authorized scope.
5. Follow the approved technology architecture.
6. Update tests and documentation.
7. Report implementation and verification evidence.

The agent must not silently change specifications or invent business rules.

### Gate 4 — Automated Compliance

GitHub Actions must validate:

- Specification integrity.
- Requirement traceability.
- ADR dependencies.
- Approved technology usage.
- API and schema compatibility.
- Unit and integration tests.
- Functional parity.
- Documentation synchronization.

Noncompliant pull requests must be blocked.

### Gate 5 — Independent Conformance Review

A separate AI agent reviews the implementation against the approved specifications.

The reviewer must verify:

- Requirement coverage.
- Contract compliance.
- Functional parity.
- Architectural compliance.
- Test coverage.
- Unapproved changes.
- Unresolved ambiguities.

The implementing agent must not self-approve architecture decisions or functional parity.

---

## 4. Recommended Repository Structure

Preserve the existing `docs/00–15`, `docs/adr/`, and `docs/prompts/` structure.

Extend it with governance, contracts, and automated verification.

```text
datacompass-crawlers/
├── AGENTS.md
├── CLAUDE.md
├── README.md
│
├── docs/
│   ├── 00-index.md
│   ├── 01-... through 15-...
│   │
│   ├── adr/
│   │   └── Existing ADRs
│   │
│   ├── prompts/
│   │   ├── 00-how-to-use.md
│   │   ├── P00-P18
│   │   └── P99-review
│   │
│   ├── governance/
│   │   ├── constitution.md
│   │   ├── definition-of-done.md
│   │   └── approval-policy.md
│   │
│   ├── requirements/
│   │   └── requirements.yaml
│   │
│   ├── contracts/
│   │   ├── canonical-entity.schema.json
│   │   ├── lineage-event.schema.json
│   │   ├── crawler-run.schema.json
│   │   ├── control-plane.openapi.yaml
│   │   └── sqlglot-boundary.md
│   │
│   ├── parity/
│   │   ├── atlan-sync-matrix.yaml
│   │   └── fixtures/
│   │
│   └── traceability.yaml
│
├── tests/
│   ├── contracts/
│   ├── parity/
│   └── integration/
│
├── scripts/
│   └── validate-specs.*
│
└── .github/
    ├── workflows/
    │   └── sdd-gate.yml
    └── CODEOWNERS
```

### Existing Documentation vs. Enforcement Artifacts

| Existing Artifact | Required Enforcement Artifact |
|---|---|
| Architecture documentation | Approved architecture baseline |
| ADRs | Machine-checkable decision statuses |
| 20-row parity matrix | Executable parity tests |
| Canonical data contract | Versioned machine-readable schema |
| Control-plane API design | OpenAPI contract |
| P00–P18 prompts | Work orders with requirement IDs |
| Coding standards | Automated policy enforcement |
| Testing strategy | Executable acceptance test suite |
| Deployment documentation | Reproducible deployment configuration |

Avoid maintaining duplicate, independently editable sources of truth.

---

## 5. Resolve Architecture Dependencies Before Coding

Several documented open questions can materially affect implementation.

| Decision | Required Action | Enforcement |
|---|---|---|
| Q-04 | Assign owner for migrating Atlan-authored protection flags and business-term links | Block destructive cutover |
| Q-05 | Define exact Unity Catalog write-back selection and ownership | Block write-back implementation |
| Q-06 | Confirm responsibility for lineage relationships | Block dependent lineage integration |
| Q-10 | Approve Databricks JDBC artifact | Block JDBC-specific implementation |
| Q-11 | Validate Python sidecar deployment on GAIA | Block sidecar integration |
| Q-17 | Obtain approved SEAL identifier | Block affected environment deployment |
| Q-19 | Confirm metadata-app/WMA responsibility boundaries | Block duplicate API implementation |

### ADR Approval Policy

Each ADR must have:

- Unique identifier.
- Decision description.
- Context and alternatives.
- Consequences.
- Owner.
- Approval status.
- Approval date.
- Related requirements.
- Dependent implementation work orders.

Supported decision statuses:

- Proposed
- Under Review
- Approved
- Rejected
- Superseded

AI agents cannot approve ADRs.

Not all 12 ADRs must necessarily be approved before any coding begins.

Each work order should declare its dependencies, and unresolved decisions should block only the affected work.

---

## 6. Convert the Parity Matrix into Executable Specifications

The existing 20-row Atlan-sync parity matrix should become a machine-verifiable contract.

### Required Classifications

Every parity row must be classified as:

| Classification | Meaning |
|---|---|
| Parity | Preserve approved existing behavior |
| Enhancement | Implement new behavior intentionally absent from the legacy system |
| Correction | Replace incorrect legacy behavior with approved expected behavior |
| Migration | Preserve or transfer existing data, metadata, or ownership |

### Example

The research indicates that the existing sync application does not map owners for tables or columns.

Therefore:

- Owner mapping must not be incorrectly classified as existing parity.
- It should be classified as a new capability.
- The new behavior requires an approved specification.
- Automated tests must validate the approved behavior.

### Required Parity Evidence

Each parity row must include:

1. Requirement ID.
2. Classification.
3. Legacy source reference.
4. Approved expected behavior.
5. Input fixtures.
6. Expected output fixtures.
7. Error and edge cases.
8. Implementation work order.
9. Acceptance test IDs.
10. Verification status.

### Golden Fixture Strategy

Use approved and sanitized legacy examples as reference fixtures.

```text
Legacy Python / TypeScript
          |
          v
  Reference Input/Output
          |
          v
  Approved Behavior Contract
          |
          v
   Java Implementation
          |
          v
  Automated Comparison
          |
          v
      Pass / Fail
```

Tests should cover:

- Normal execution.
- Invalid inputs.
- Empty and null values.
- Metadata changes.
- Deleted entities.
- Lineage transformations.
- Permission boundaries.
- Idempotency.
- Failure and retry handling.
- Data consistency.

Do not automatically treat every legacy output as correct.

Known defects require an approved corrected behavior contract.

---

## 7. Mandatory Architecture Constitution

Create:

`docs/governance/constitution.md`

The constitution should enforce these principles.

### Principle 1 — Specification Authority

Approved specifications are authoritative for required system behavior.

Implementation must conform to approved specifications.

### Principle 2 — Java-First Architecture

The primary application must use Java.

Java owns:

- Crawler orchestration.
- Connector SPI.
- Execution lifecycle.
- Run-state management.
- Canonical entity persistence.
- Control-plane APIs.
- Operational error handling.

### Principle 3 — Restricted Python Usage

Python is allowed only within the explicitly approved SQLGlot integration boundary.

Python must not implement:

- Main crawler orchestration.
- Connector framework.
- Control plane.
- Canonical persistence.
- Run-state management.

Python sidecar deployment remains dependent on Q-11 approval.

### Principle 4 — Contract-First Implementation

Interfaces and contracts must be approved before dependent implementations.

This applies to:

- Connector SPI.
- Canonical metadata model.
- Lineage contracts.
- Crawler run lifecycle.
- Control-plane APIs.
- SQLGlot interoperability.

### Principle 5 — No Undocumented Behavior

Agents may not silently introduce:

- Business rules.
- API fields.
- Database schema changes.
- Lifecycle states.
- Metadata placeholder values.
- Integration behavior.
- Security permissions.

### Principle 6 — No Unauthorized Architecture Changes

Architecture changes require an ADR.

The relevant ADR must be approved before the change is implemented.

### Principle 7 — Specification Synchronization

Any change to observable behavior must update its corresponding specifications and tests.

### Principle 8 — Independent Verification

Implementation completion does not mean specification compliance.

Verification requires objective evidence and independent review.

### Principle 9 — Rebuildability

The repository must contain sufficient approved specifications and supporting artifacts to reconstruct the application independently.

### Principle 10 — Traceability

Every implemented requirement must trace to:

```text
Requirement
    |
    v
Source Evidence
    |
    v
Approved Decision
    |
    v
Design Contract
    |
    v
Implementation Work Order
    |
    v
Source Code
    |
    v
Automated Acceptance Test
    |
    v
Verification Evidence
```

---

## 8. Work-Order Lifecycle for P00–P18

Each work order follows a controlled lifecycle.

```text
Draft
  |
  v
Reviewed
  |
  v
Approved
  |
  v
Implemented
  |
  v
Tested
  |
  v
Verified
```

Failed verification returns the work order to implementation or specification review.

### Mandatory Execution Sequence

For each implementation work order:

1. Load approved specifications.
2. Identify requirement IDs.
3. Validate ADR dependencies.
4. Validate interface contracts.
5. Prepare acceptance tests.
6. Implement only the approved scope.
7. Execute required automated tests.
8. Update implementation traceability.
9. Run an independent conformance review.
10. Obtain required human approval before merging.

### P99 Independent Review

Strengthen the existing P99 review prompt to generate a structured conformance report.

The report must identify:

- Requirements satisfied.
- Requirements partially satisfied.
- Requirements not implemented.
- Architecture violations.
- Contract violations.
- Legacy parity failures.
- Missing test coverage.
- Unauthorized functionality.
- Documentation drift.
- Outstanding risks and decisions.

Use a separate agent session for independent review.

---

## 9. GitHub Enforcement

Agent instructions alone are insufficient.

The repository must enforce compliance through GitHub Actions and branch protection.

### Required Repository Controls

- Protected main branch.
- Required pull requests.
- Mandatory status checks.
- CODEOWNERS reviews.
- Spec validation checks.
- Automated contract tests.
- Architecture tests.
- Functional parity tests.
- Security and dependency checks.

### Proposed CI Checks

| Check | Pass Condition |
|---|---|
| Specification integrity | Valid IDs, references, and required fields |
| Requirement traceability | Changed behavior maps to approved requirements |
| ADR dependencies | Required decisions are approved |
| Technology restrictions | Unauthorized Python application code is absent |
| Contract validation | API and schema checks pass |
| Parity verification | Required reference fixtures pass |
| Java quality | Build, unit tests, and static analysis pass |
| Integration verification | Required integration tests pass |
| Documentation synchronization | Behavior changes include specification and test updates |

Use architecture tests, such as ArchUnit, to enforce Java module boundaries.

Use a narrowly controlled human-reviewed exception process for nonbehavioral refactoring and other legitimate changes.

---

## 10. Definition of Done

A feature is not complete merely because Claude or Copilot generated functioning code.

### Mandatory Definition of Done

- [ ] Requirement IDs exist.
- [ ] Relevant specifications are approved.
- [ ] Blocking ADR dependencies are approved.
- [ ] API and data contracts are finalized.
- [ ] Required acceptance tests exist.
- [ ] Implementation conforms to approved architecture.
- [ ] Unit tests pass.
- [ ] Integration tests pass.
- [ ] Applicable parity tests pass.
- [ ] Security checks pass.
- [ ] Specifications reflect actual intended behavior.
- [ ] Traceability mappings are updated.
- [ ] Independent conformance review is completed.
- [ ] Required human approvals are obtained.
- [ ] Changes are merged through the protected branch.

---

## 11. Application Reconstruction Readiness

The long-term objective is to rebuild the application from specifications without relying on the original implementation.

### Clean-Room Rebuild Procedure

1. Create an isolated development environment.
2. Provide only the approved specifications and supporting artifacts.
3. Exclude the original application source implementation.
4. Give the new agent the approved contracts, acceptance tests, deployment definitions, and permitted dependencies.
5. Rebuild the application.
6. Run the same independent acceptance suite.
7. Compare behavior with approved reference fixtures.
8. Document specification ambiguities and implementation gaps.
9. Correct the specifications.
10. Repeat until required reconstruction criteria pass.

### Reconstruction Readiness Metrics

| Metric | Proposed Target |
|---|---|
| Requirement verification coverage | 100% of approved requirements |
| Required parity coverage | 100% |
| Contract validation coverage | 100% of required contracts |
| Implementation traceability | 100% |
| Unresolved release-blocking decisions | 0 |
| Critical architecture violations | 0 |
| Required acceptance test failures | 0 |

These targets do not themselves prove rebuildability.

An independently reconstructed implementation passing the acceptance suite is the strongest verification.

### Important Distinction

**Functional reconstruction** means recreating the application's required functionality from specifications.

**Disaster recovery** requires additional capabilities, including:

- Production data backups.
- Source and artifact backups.
- Infrastructure definitions.
- Environment configuration.
- Secrets recovery procedures.
- Deployment automation.
- External assets.

Specifications alone cannot restore production customer data.

---

## 12. Recommended Implementation Sequence

| Priority | Action | Outcome |
|---|---|---|
| 1 | Commit the current documentation as a draft baseline | Reproducible starting point |
| 2 | Perform specification completeness audit | Gaps and contradictions identified |
| 3 | Review ADRs and assign open-question owners | Approval dependencies established |
| 4 | Formalize the 20-row parity matrix | Testable legacy behavior |
| 5 | Create machine-readable contracts | Enforceable interface definitions |
| 6 | Establish governance and CI controls | Specification enforcement |
| 7 | Execute P00 read-only | Verify project assumptions |
| 8 | Begin approved P01 scaffolding | Controlled implementation |
| 9 | Execute P02–P18 with independent reviews | Continuous conformance |
| 10 | Perform clean-room reconstruction tests | Validate rebuildability |

---

# 13. Prompt for the Specification Governance Agent

Copy the following into Claude Code or another AI agent responsible for hardening the existing specifications.

## Role

You are the **Specification Governance Architect** for the `datacompass-crawlers` repository.

## Objective

Convert the existing design documentation and implementation prompt pack into an enforceable Spec-Driven Development system.

The resulting specification system must allow a separate AI agent to rebuild the application from approved specifications without relying on undocumented knowledge of the legacy implementation.

This is a governance and specification-hardening task.

**Do not implement application functionality.**

## Existing Artifacts

Preserve and inspect:

- `README.md`
- `AGENTS.md`
- `CLAUDE.md`
- `docs/00–15`
- `docs/adr/`
- `docs/prompts/`, including P00–P18 and P99
- The 20-row Atlan-sync parity matrix
- Existing open questions
- Proposed architecture decisions

Do not replace existing specifications or silently change their meaning.

## Mandatory Architecture Rules

1. The primary application must be implemented in Java.
2. Python is permitted only within a clearly isolated SQLGlot integration boundary.
3. Python must not implement main crawler orchestration, connector SPI, control plane, canonical persistence, or run-state management.
4. Python sidecar deployment is conditional on approval of Q-11.
5. Reuse the approved Java application and platform architecture.
6. Preserve required existing behavior from the Python and TypeScript implementations.
7. Document deviations from legacy behavior as approved enhancements or corrections.
8. Do not invent architecture decisions, business rules, or answers to open questions.
9. Do not introduce undocumented database fields, API responses, lifecycle states, or placeholder metadata.
10. Preserve ownership of metadata-app read APIs unless a boundary change is approved.

## Required Deliverables

### A. Specification Governance

Create:

- Project constitution.
- Specification lifecycle.
- Approval policy.
- Definition of Done.
- Change-control process.

Define specification statuses:

- Draft
- In Review
- Approved
- Implemented
- Verified
- Superseded

An AI agent must not independently approve business or architecture specifications.

### B. Traceability

Establish machine-readable traceability connecting:

```text
Requirement
  -> Source Evidence
  -> Approved Decision
  -> Design Contract
  -> Work Order
  -> Implementation
  -> Automated Tests
  -> Verification Evidence
```

Every parity row must have:

- Unique identifier.
- Classification.
- Acceptance criteria.
- Verification strategy.
- Implementation mapping.

### C. Machine-Readable Contracts

Identify and formalize:

- Canonical entity schema.
- Lineage event schema.
- Crawler run lifecycle.
- Connector SPI.
- Control-plane APIs.
- Java/Python SQLGlot boundary.

Contracts must specify:

- Data types.
- Validation rules.
- Failure behavior.
- Versioning.
- Compatibility requirements.

### D. Agent Instructions

Update `AGENTS.md` and `CLAUDE.md`.

Require all implementing agents to:

1. Read relevant approved specifications before coding.
2. Identify applicable requirement IDs.
3. Check ADR dependencies.
4. Identify unresolved decisions.
5. Stop affected implementation when requirements are ambiguous.
6. Avoid unauthorized functionality.
7. Update specifications and tests with implementation changes.
8. Produce a completion and conformance report.

### E. Automated Validation

Create an SDD validation mechanism suitable for GitHub Actions.

Checks should include:

- Requirement ID validity.
- Traceability integrity.
- Approval dependencies.
- Specification synchronization.
- Test references.
- Schema validation.
- OpenAPI validation.
- Architecture constraints.
- Documentation link integrity.

Do not claim that textual validation proves functional parity.

### F. Independent Review

Define a workflow in which a separate AI agent reviews completed implementation work.

The implementing agent must not self-approve:

- Architecture decisions.
- Requirement exceptions.
- Functional parity.
- Specification compliance.

### G. Reconstruction Readiness

Define a clean-room rebuild process.

The rebuild agent must be able to implement the application using only:

- Approved specifications.
- Architecture contracts.
- Machine-readable schemas.
- Acceptance tests.
- Approved dependencies.
- Infrastructure definitions.
- Required test fixtures.

Do not allow access to the original application implementation during the independent reconstruction test.

## Completion Requirements

Produce a final report containing:

1. Files created or modified.
2. Existing documentation preserved.
3. Specification gaps discovered.
4. Contradictions identified.
5. Unresolved decisions and affected work orders.
6. Automated validations implemented.
7. Actual validation results.
8. Remaining manual approvals.
9. Recommended execution order for P00 and P01.
10. Reconstruction-readiness assessment.

Do not begin application coding.

Do not approve ADRs.

Do not modify unrelated repositories.

---

# 14. Final Recommendation

Adopt the following operating model:

```text
       Legacy Repositories
       Python / TypeScript
                |
                v
        Verified Research
                |
                v
       Approved Specifications
                |
                v
       Machine-Readable Contracts
                |
                v
       Automated Acceptance Tests
                |
                v
       Claude / Copilot Coding
                |
                v
       CI Compliance Validation
                |
                v
       Independent AI Review
                |
                v
       Human Approval
                |
                v
       Versioned Release
```

For DataCompass Crawlers, prioritize these technical contracts:

1. Canonical metadata data model.
2. Connector SPI.
3. Crawler run lifecycle.
4. SQLGlot interoperability boundary.
5. Lineage events and transformations.
6. Control-plane API.
7. Atlan-sync parity behavior.
8. Deployment and operational contracts.

## Conclusion

The immediate next action should **not** be P01 implementation.

First, convert the existing documentation, ADRs, parity matrix, and work orders into a governed and testable specification baseline.

Once established:

- Specifications define required behavior.
- Contracts define integration boundaries.
- Acceptance tests verify functionality.
- AI agents generate implementation.
- CI prevents nonconforming changes.
- Independent reviews verify compliance.
- Human owners approve decisions.
- Clean-room rebuilds validate specification completeness.

**The ultimate objective is to make the specifications, contracts, and acceptance tests the permanent assets of DataCompass Crawlers, while keeping the implementation replaceable.**