Recommendation: Do not copy all docs verbatim.

Use fresh docs tailored to the Data Compass Control Plane, while reusing the relevant patterns from datacompass-ui as source guidance.

The new repo should have its own focused guardrail docs because the control plane has different responsibilities: workflow administration, crawler/miner run management, configuration versioning, run history, logs, audit, and orchestration adapters.

Recommended approach:

- Generate fresh control-plane-specific docs.
- Reuse the same UI/BFF/API patterns from datacompass-ui.
- Reference datacompass-ui docs only where helpful.
- Copy only the truly shared guardrails, not the full 851k-word doc set.
- Add clear docs for control-plane architecture, routing, API usage, generated types, UI page patterns, RBAC, audit, and workflow/run concepts.

Final decision:

Choose option 3, or a custom hybrid: generate fresh docs tailored to the control plane and reference datacompass-ui patterns as needed. Avoid copying all docs verbatim because it will bloat the repo and may confuse agents with guidance that belongs to the original app, not the control plane.