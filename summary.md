Review the current `datacompass-crawlers` specification-driven development baseline.

The objective is to establish a production-grade, enforceable SDD process that allows the application to be rebuilt from scratch using approved specifications, without depending on the original implementation or undocumented agent knowledge.

Before implementing application functionality:

1. Audit the existing specifications, ADRs, contracts, work orders, and AGENTS.md for consistency, completeness, duplication, and unresolved dependencies.

2. Implement the `crawler-spec` Maven module so specification validation executes as part of the standard `mvn clean install` lifecycle. Do not rely on a temporary or session-local checker.

3. Enforce protected acceptance-test and golden-fixture paths. Implementation changes must not modify these artifacts without a separate human-approved change.

4. Establish traceability from requirement → architectural decision → contract → work order → acceptance test → implementation.

5. Ensure golden fixtures are derived from pinned legacy implementations and representative data, with documented provenance and deterministic normalization.

6. Validate positive, negative, failure, retry, partial-write, idempotency, and recovery scenarios where applicable.

7. Maintain the Java-first architecture, using Python only for the approved SQLGlot sidecar. Define and test the boundary contract.

8. Preserve existing human approval requirements. Do not automatically approve ADRs, parity classifications, requirements, or unresolved design decisions.

9. Generate a structured conformance report showing passed checks, failures, uncovered requirements, unresolved decisions, and implementation readiness.

10. Define a repeatable clean-room rebuild procedure that can be executed by an independent agent using only approved specifications, fixtures, contracts, and declared build dependencies.

Do not rewrite or replace existing specifications unnecessarily. Extend the current structure and reuse existing requirement IDs.

Do not begin major application implementation until the specification enforcement framework is operational.

At completion, provide:
- Files created or modified
- Validation checks implemented
- Checks actually executed and their results
- Remaining gaps and blockers
- Decisions requiring human approval
- Recommendation on whether P01/P02 implementation can proceed

Do not claim a check passed unless it was actually executed successfully.