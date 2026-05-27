# Need to Know — JRN Minting Service and Dual-Write

These are the two things blocking the load layer from being fully specified. Here's exactly what's needed and why each matters.

---

## JRN Minting Service

| Question | Why it matters |
|---|---|
| **Interface** — is it a REST API, a library/SDK, a message-based service? What's the call signature? | Determines how the normalizer integrates it — an injected HTTP client vs. an imported library changes the code and the failure modes. |
| **Idempotency** — does minting twice for the same Databricks object return the same `jrn`? | If not idempotent, every re-crawl creates duplicate catalog entries and orphans relationships. This is the single most important question. |
| **Input contract** — what does it need to mint a `jrn`? Platform + catalog + schema + object name? Something else? | The normalizer must supply exactly the right inputs; the interface contract's `mint_distribution_jrn(...)` signature is a guess until confirmed. |
| **Parent/child** — how does a `DataAttribute` jrn reference its parent `DataDistribution` jrn? Does the service handle the hierarchy, or does the crawler pass the parent jrn in? | Determines whether attribute minting is one call or two, and whether the crawler must mint the parent first. |
| **Rename behavior** — if a Databricks table is renamed, does the service mint a new `jrn` or return the existing one? | Decides whether a rename is "same asset, new name" or "delete + create" — this changes the delta logic and the history collections. |
| **Authentication & availability** — how does the crawler authenticate to it? What's its SLA / what happens when it's down mid-crawl? | The crawler can't write a document without a `jrn`; if the service is unavailable, the run must fail cleanly and resume, not write `jrn`-less records. |
| **Existing jrn format** — can you get a few sample `jrn` values from existing catalog documents? | Lets us verify the new crawler produces `jrn`s consistent with what's already there — critical so cutover doesn't orphan existing relationships. |

---

## Dual-Write Contract

| Question | Why it matters |
|---|---|
| **Who reads `atlan.*` today?** The internal UI, MongoDB indexes, reports, other services? | The dual-write window exists to protect these readers. We need the list to know when `atlan.*` can be safely dropped. |
| **Field-level mapping confirmation** — is the `atlan.*` → `source.*` mapping in the parity matrix §2 correct and complete? | The load layer dual-writes both blocks; any missed field regresses a reader. |
| **MongoDB indexes on `atlan.*`** — are there indexes on `atlan.guid` or other `atlan.*` paths? | Indexes must be recreated on `source.*` before readers migrate; this is a coordination item with whoever owns the DB. |
| **Transition exit criteria** — what defines "all readers migrated, drop `atlan.*`"? Who signs off? | Without an explicit exit criterion the dual-write window never closes and the technical debt is permanent. |
| **Write semantics** — during dual-write, are `atlan.*` and `source.*` always identical values, or can they diverge? | They should be identical (same Databricks-sourced values). Confirming this rules out a class of subtle bug. |



## What still needs your platform team: 

the actual Jules pipeline registration, the Managed Repos / Artifactory PyPI repository provisioning, builder-node setup, and the Kubernetes-vs-ECS deployment target (still unresolved, still E0.4).

1. Interface — Is the JRN minting service exposed as a REST API, an importable library/SDK, or a message-based service? What is the call signature for minting a Distribution (table/view) JRN and an Attribute (column) JRN?

2. Idempotency — If we call the minting service twice with the same inputs (platform + catalog + schema + object name), will we always get the same JRN back? This is critical — non-idempotent minting would cause duplicate catalog entries on every re-crawl.

3. Input contract — What inputs does the service require to mint a Distribution JRN? We expect something like: platform (DATABRICKS), workspace name, catalog name, schema name, and object name. Please confirm the exact required fields.

4. Parent/child relationship — For minting an Attribute (column) JRN, does the service handle the parent-child hierarchy internally, or does the caller pass the parent Distribution JRN as an input? We need to know whether to mint the parent first and pass it in, or whether the service derives it.

5. Rename behavior — If a Databricks table is renamed at source, should we mint a new JRN for the renamed object, or does the service return the existing JRN for that asset? This determines whether a rename is treated as "same asset, new name" or "delete + create new" in the catalog.

6. Authentication and availability — How does a service call authenticate to the JRN minting service? And what is the expected behavior when the service is temporarily unavailable mid-crawl — should we fail the crawl run and retry, or is there a fallback?
