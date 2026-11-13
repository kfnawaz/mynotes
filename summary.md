## Tableau SDR – Meeting Summary

### Objective
Validate Tableau metadata extraction and ingestion into Atlan through SDR, including JWT authentication and secure credential retrieval from GSM/HashiCorp Vault.

### Blockers
- **SDR Runtime:** SDR orchestrator/Tableau application issues prevented live testing.
- **Vault Integration:** GSM supports only one value per secret, requiring additional configuration and validation.
- **UI Issues:** Duplicate credential fields and inconsistent labels (*Path* vs. *Key Name*) create configuration ambiguity.

### Key Findings
- **Two-phase validation:** First validate SDR using direct JWT credentials, then test GSM/HashiCorp Vault integration.
- **SDR Testing:** Direct JWT credentials can validate Tableau connectivity, metadata extraction, transformation, and ingestion into Atlan without Vault dependency.
- **GSM Limitation:** Four separate secrets are required. Atlan recommended *Single Key* mode with individual secret references.
- **UI Issues:** Atlan acknowledged duplicate fields and inconsistent labeling.

### Action Items
1. **Internal Team:** Restore SDR runtime and test with direct JWT credentials over the weekend.
2. **Internal Team:** Configure four GSM secrets and validate Vault-based credential retrieval.
3. **Atlan:** Report UI inconsistencies and assist with Vault path/key mapping.
4. **Both Teams:** Reconvene Monday morning for joint testing and troubleshooting.

### Conclusion
**Agreed Approach:** First prove SDR works independently using direct credentials, then validate GSM Vault integration. Both remain unverified pending testing.