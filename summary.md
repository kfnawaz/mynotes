Recommendations for the open decisions:

1. Is WMA a new repo or does it extend server?

Recommendation: New repo / separate backend service.

The WMA should not extend the existing Fastify BFF/server. The BFF should remain auth/config/proxy focused. The WMA is a real backend service with workflow APIs, MongoDB access, run history, orchestration integration, normalization, and audit responsibilities. Keeping it separate avoids increasing the blast radius of the auth layer.

Final decision:
- Create WMA as a separate backend service/repo.
- Keep BFF clean: auth/session/config/proxy only.
- SPA calls WMA through the existing BFF/API routing pattern if required.

2. Does WMA read runSummaries directly or maintain its own workflow_runs collection?

Recommendation: Start with direct read + normalization from Harvester runSummaries, but WMA should maintain its own normalized operational view.

The Harvester’s runSummaries collection can be the source acquisition point for pilot integration, but the WMA should normalize that into its own control-plane model for UI, filtering, audit, retries, version correlation, and long-term run history.

Final decision:
- Slice 1: WMA reads Harvester runSummaries.
- WMA normalizes into workflow_runs / workflow_run_steps / workflow_run_events where needed.
- UI reads only from WMA APIs, not directly from Harvester MongoDB collections.
- Longer term, WMA owns the UI-facing run history model.

3. snake_case gap: should Harvester upgrade runSummaries to camelCase or should WMA normalize on read?

Recommendation: Ask Harvester to emit camelCase in runSummaries for Slice 1 if feasible.

Since the Harvester contract/schema already appears to target camelCase, the cleanest path is for runSummaries to match that contract. This reduces permanent mapping logic and avoids two naming conventions across the control-plane boundary.

Final decision:
- Preferred: Harvester writes camelCase runSummaries.
- Fallback: WMA normalizes snake_case to camelCase at the boundary.
- Do not let snake_case leak into SPA/UI models.

4. run_type → runMode mapping

Recommendation: Normalize at the WMA boundary and define a controlled mapping.

Harvester terms like "crawl" / "lineage" describe workflow category/type. SPA terms like "FULL" / "VALIDATE" describe execution mode. These are different concepts and should not be overloaded.

Final decision:
- Use workflowType for CRAWLER / MINER / INDEXER / VALIDATION.
- Use runMode for FULL / INCREMENTAL / VALIDATE_ONLY / DRY_RUN / LIMITED_SCOPE.
- Map Harvester run_type only to workflowType, not runMode.
- WMA owns the boundary mapping.

Example:
- run_type = "crawl" → workflowType = "CRAWLER"
- run_type = "lineage" → workflowType = "MINER"
- runMode should come from the execution request/config, not from run_type.

5. Who runs the WMA in dev?

Recommendation: WMA should run as a separate dev process owned by the WMA repo.

Do not hide it inside the SPA/BFF dev server. Developers should be able to start the SPA, BFF, and WMA independently.

Final decision:
- WMA runs as its own backend process.
- Dev setup can provide a convenience script to start all processes together.
- Example:
  - npm run dev:ui
  - npm run dev:bff
  - npm run dev:wma
  - npm run dev:all

This keeps ownership clean while still making local development easy.