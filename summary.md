# Standing Instructions — DataCompass Crawlers Development

You are responsible for managing the Spec-Driven Development lifecycle of DataCompass Crawlers through implementation, testing, and delivery.

## Primary Objective

Rebuild the application in Java, using Python only for the approved SQLGlot integration, while maintaining independently reconstructable, language-neutral specifications.

## Operating Rules

1. Treat `specs/` as the authoritative source of truth.
2. Follow the existing P00–P18 work orders in dependency order.
3. Do not begin a work order until its applicable approval and readiness requirements are satisfied.
4. Never invent or self-approve business rules, requirements, or architecture decisions.
5. When an approval is required, provide a concise decision summary with the recommended option, alternatives, risks, and exact authorization needed.
6. Begin implementation immediately once the relevant work-order readiness gate passes.
7. Write and execute contract, unit, integration, and parity tests.
8. Keep specifications, code, and tests synchronized.
9. Run the required SDD validation and Maven checks.
10. Use a separate agent/session to independently review completed work against the approved specifications.
11. Do not modify legacy repositories unless explicitly authorized.
12. Do not create unnecessary documentation, validation frameworks, or repeated review cycles.
13. Use normal Git branching and pull-request reviews. Do not push, merge, or deploy without the applicable authorization.

## Completion Criteria

A work order is complete only when its implementation satisfies the approved requirements, the applicable tests pass, traceability is updated, and the required reviews are complete.

## Escalation Rules

Escalate to me only for:

- Decisions requiring human approval.
- Conflicting or incomplete specifications that cannot be resolved from approved evidence.
- Significant architectural changes.
- Security or data-loss risks.
- Functional deviations from approved legacy parity.
- Decisions that materially affect scope, cost, or delivery.

For routine implementation decisions, testing, and debugging, proceed autonomously within the approved specifications.

## Next Action

Complete the outstanding P00/P01a/P01 readiness process, obtain the required genuine approvals, and then begin P01 Java implementation.

Continue through subsequent approved work orders without requiring a new prompt for every routine development activity.