# SDR Adoption Planning Discussion

## Overall Migration Strategy

The team plans to adopt **SDR (Secure Data Runtime)** as the new architecture, which will effectively replace the current Secure Agent deployment.

This is **not just an upgrade**, but a completely new deployment model that requires:

- Procurement of new container images for each connector.
- Deployment of the SDR components, including the **SDR Orchestrator**.
- Rebuilding the deployment using the new SDR architecture.

---

# Image Management & Updates

## Initial Image Procurement

All required connector images (Snowflake, Tableau, Databricks, ThoughtSpot, etc.) must be imported into JPMorgan Chase's internal container repository.

## Ongoing Image Updates

Since JPMC uses an **internal image repository** instead of the vendor's registry:

- The SDR Orchestrator cannot automatically pull newly released images.
- A separate automation should:
  - Periodically check the vendor image registry.
  - Detect newly released images.
  - Pull those images into the internal JPMC repository.

Once images are available internally, the SDR Orchestrator can use them for deployments.

---

# Recommended Deployment Model

The recommendation is to deploy SDR using **Docker** rather than Kubernetes.

### Reasons

- Reuses the existing VM infrastructure.
- Simpler operational model.
- No dependency on Kubernetes administrators.
- Easier for application teams to manage.
- Supports:
  - Local storage
  - Enterprise secret managers
  - Existing VM environments

Kubernetes remains supported but is not considered necessary for the current deployment.

---

# Multi-Environment Deployment

Running multiple environments on the same VM is considered feasible.

Examples:

- Production
- UAT
- Sandbox
- Experimental

Each environment can have:

- Separate Docker containers
- Separate configuration folders
- Separate registration endpoints
- Independent connector configuration

This provides isolation similar to Kubernetes namespaces while remaining operationally simpler.

---

# Recommended Migration Sequence

The suggested rollout approach is:

1. Deploy the SDR infrastructure.
2. Configure enterprise secret management.
3. Migrate a single connector first (recommended: Tableau).
4. Validate the deployment.
5. Add additional connectors one at a time:
   - Snowflake
   - Databricks
   - ThoughtSpot
   - Others

This minimizes migration risk and simplifies troubleshooting.

---

# Infrastructure Strategy

Rather than continuing to invest in the existing VM-based deployment, the discussion suggested evaluating a **cloud-first deployment**.

Since the organization's long-term direction is AWS:

- Consider deploying SDR directly into AWS.
- Build the new platform on the target infrastructure instead of the legacy VM.

Potential benefits include:

- Better scalability
- Cleaner architecture
- Easier long-term maintenance
- Alignment with future infrastructure strategy

---

# Secrets Management

SDR requires integration with an enterprise secret management solution.

Requirements:

- Credentials should never be embedded inside Docker images.
- Docker containers should retrieve secrets securely during runtime.
- The hosting VM or cloud infrastructure must have permission to access the selected secret manager.

---

# Networking & Connectivity

One of the most important architectural discussions centered around networking.

## Current State

Current communication relies on:

- Enterprise proxy servers
- Proxy exceptions
- Annual security approvals
- Architecture exceptions

These add operational complexity.

## Proposed Future State

Investigate using:

- Private networking
- Private endpoints
- AWS PrivateLink (or equivalent private connectivity)

Advantages:

- Removes dependency on enterprise proxies.
- Eliminates recurring security exceptions.
- Simplifies connectivity to cloud platforms such as:
  - Snowflake
  - Databricks
- Improves overall architecture.

The recommendation is to validate this with the enterprise networking and security teams before finalizing the architecture.

---

# Key Action Items

## Architecture

- Confirm Docker as the deployment model.
- Decide whether to deploy on existing VMs or directly in AWS.

## Images

- Procure all required SDR images.
- Build automation to synchronize images into the internal repository.

## Secrets

- Select the enterprise secret manager.
- Configure secure runtime access.

## Migration

- Begin with the Tableau connector.
- Validate the platform.
- Incrementally migrate remaining connectors.

## Networking

- Investigate replacing proxy-based connectivity with private networking.
- Engage networking and security teams early.
- Eliminate annual proxy exception processes where possible.

---

# Overall Conclusion

The migration to SDR should be viewed as a **greenfield architectural implementation**, not simply a replacement of the Secure Agent.

Before implementation begins, several foundational architectural decisions should be finalized:

- Deployment platform (VM vs AWS)
- Docker deployment strategy
- Image lifecycle management
- Secrets management
- Private networking architecture
- Migration sequencing

Addressing these items first will reduce operational complexity and provide a cleaner, more maintainable SDR deployment.