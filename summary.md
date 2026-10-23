# Meeting Summary: SDR Deployment and Secrets Management Strategy

## 1. Objective

Determine a **secure, technically feasible, and supportable approach for deploying Atlan's Secure Data Runtime (SDR)** on the existing VSI infrastructure, while resolving authentication and secrets management challenges.

The primary goal is to replace the existing Secure Agent architecture without introducing unacceptable security risks, unsupported Kubernetes dependencies, or significant product changes.

---

## 2. Key Points Discussed

### 2.1 AWS Secrets Manager – Security Restrictions

- The AWS security/architecture team indicated that accessing AWS Secrets Manager from a runtime outside the AWS VPC is likely prohibited by a hard security control.
- The concern is primarily the architecture pattern, not the technical implementation of IAM/SAML authentication.
- An exception may be possible, but the team was directed to consult IAM product managers.
- Moving SDR into AWS would resolve the network-boundary concern, but AWS deployment approval remains pending.

### 2.2 Existing K3s Infrastructure

- The existing K3s environment already contains Kubernetes Secrets and token-generation CronJobs.
- Token generators refresh credentials approximately every 30 minutes.
- SDR could technically run in K3s, potentially in a separate namespace.
- However, Atlan strongly discouraged this approach because of:
  - Previous operational issues.
  - Limited Kubernetes expertise.
  - Unclear long-term support ownership.
  - Additional dependencies on the existing K3s infrastructure.

### 2.3 Local Secrets and Dynamic Refresh

- SDR uses Dapr components and can integrate with multiple secret stores, including local file-based stores.
- Existing token-generation jobs could potentially run directly on VSI without Kubernetes.
- Updating environment variables would not automatically refresh credentials inside running Docker containers; a restart would generally be required.
- File-based credentials may offer an alternative, subject to verifying whether SDR can consume updates without restarting.
- Storing credentials on local disk introduces security concerns requiring approval.

### 2.4 GSM / HashiCorp Vault Integration

- GSM was discussed as a modified HashiCorp Vault-based secrets service.
- Dapr already supports HashiCorp Vault as a secret store.
- The principal challenge appears to be GSM's authentication mechanism, rather than secrets retrieval itself.
- A lightweight authentication proxy or helper process could potentially bridge SDR and GSM without requiring significant SDR product changes.
- Atlan agreed to investigate the feasibility of providing such a component.

---

## 3. Options Considered

| Option | Feasibility | Challenges | Assessment |
|--------|-------------|------------|------------|
| **1. Deploy SDR in AWS** | Technically feasible | AWS migration approval pending | Preferred long-term architecture |
| **2. Local file-based secrets on VSI** | Technically feasible | Security approval and dynamic refresh behavior | Potential lowest-effort near-term solution |
| **3. GSM via authentication proxy** | Requires validation | Proxy development, authentication integration, support | Promising alternative |
| **4. Deploy SDR in existing K3s** | Technically feasible | Operational complexity, limited support expertise | Discouraged by Atlan |
| **5. AWS Secrets Manager from VSI** | Technically possible | Likely violates security boundary | Requires security exception |
| **6. Environment variables refreshed by CronJob** | Partially feasible | Running containers do not automatically reload variables | Not viable without restart mechanism |

---

## 4. Next Steps and Action Items

| # | Action Item | Owner | Expected Outcome |
|---|-------------|-------|------------------|
| 1 | Validate whether SDR/Dapr can dynamically read updated secrets from local files without restarting containers | Atlan team | Confirm local file-based approach |
| 2 | Check with Kailash/security whether storing secrets locally on VSI is permitted or can receive an exception | Internal team | Security approval or rejection |
| 3 | Evaluate GSM authentication integration through a lightweight proxy/helper component | Atlan + internal team | Feasibility, design, and effort estimate |
| 4 | Review GSM documentation and supported authentication mechanisms | Atlan team | Identify integration requirements |
| 5 | Investigate Podman/Docker-supported secrets mechanisms as potential alternatives | Atlan team | Determine whether a native container solution exists |
| 6 | Confirm AWS deployment strategy and pending approval | Internal cloud/architecture team | Establish cloud deployment timeline |
| 7 | Review the available approaches with Kailash and obtain direction on which solution to pursue | Internal team / Kailash | Select preferred implementation path |
| 8 | Estimate implementation effort, dependencies, and timelines for shortlisted options | Both teams | Actionable delivery plan |

---

## 5. Decisions and Agreements

1. **No final architecture was selected.**
2. AWS deployment is considered the ideal long-term solution, pending organizational approval.
3. Local file-based secrets are a potentially simpler near-term solution, provided security and dynamic refresh requirements can be satisfied.
4. GSM integration through a proxy is worth investigating because it could reuse existing Dapr capabilities.
5. Further dependency on K3s should be avoided where practical because of support and ownership concerns.
6. Both teams agreed to collaborate on technical validation rather than requiring either team to solve the integration independently.
7. Kailash's direction is needed before committing implementation resources.

---

## 6. Recommended Immediate Plan

### Step 1: Confirm Security Constraints

**Owner:** Internal team / Kailash / Security

- Determine whether local file-based secrets on VSI are permitted.
- Confirm whether AWS Secrets Manager access from VSI is categorically prohibited.
- Identify whether security exceptions are possible.

**Expected Outcome:** Clear understanding of approved secrets management patterns.

### Step 2: Validate Local File-Based Secrets

**Owner:** Atlan team

- Verify whether Dapr can retrieve updated secrets from local files dynamically.
- Determine whether SDR containers need to restart after credential rotation.
- Evaluate security and operational implications.
- Identify whether existing token-generation jobs can be reused outside K3s.

**Expected Outcome:** Confirmation of whether local file-based secrets provide a viable near-term solution.

### Step 3: Evaluate GSM Authentication Proxy

**Owner:** Atlan team + Internal team

- Review GSM authentication requirements.
- Investigate implementing a lightweight authentication proxy/helper.
- Determine whether existing Kerberos authentication and token-generation mechanisms can be reused.
- Confirm compatibility with Dapr's HashiCorp Vault secret store integration.
- Estimate development and operational support effort.

**Expected Outcome:** Technical feasibility and implementation estimate for GSM integration.

### Step 4: Obtain Architecture Direction

**Owner:** Kailash / Internal architecture team

- Compare the technically feasible and security-approved approaches.
- Assess each option based on:
  - Security compliance.
  - Implementation complexity.
  - Delivery timeline.
  - Operational supportability.
  - Long-term architectural alignment.
- Select the preferred implementation approach.

**Expected Outcome:** Approved architecture and implementation direction.

---

## 7. Overall Takeaway

The main blocker is **not SDR's ability to integrate with secret stores; it is identifying an approved authentication and secrets management architecture for an SDR runtime hosted on VSI.**

The discussion narrowed the most practical near-term paths to:

1. **Local file-based secrets** — potentially the simplest implementation, subject to security approval and dynamic credential refresh validation.
2. **GSM integration through an authentication proxy** — a promising alternative that could leverage existing Dapr capabilities while addressing GSM authentication requirements.

**AWS deployment remains the preferred long-term solution**, but depends on pending organizational decisions.

### Immediate Priority

Resolve the security questions, validate both technical approaches, and obtain **Kailash's decision on which implementation path to pursue** before committing development resources.

----------------------------------

# Executive Summary – SDR Deployment & Secrets Management

## Objective
Identify a secure and supportable approach to deploy Atlan SDR on VSI, replacing the existing Secure Agent while addressing secrets management and authentication challenges.

## Key Discussion Points
- **AWS Secrets Manager:** Access from VSI is likely restricted by security controls. Deploying SDR in AWS is preferred but pending organizational approval.
- **Local File-Based Secrets:** Potentially the simplest near-term solution, subject to security approval and validation of dynamic credential refresh.
- **GSM Integration:** A lightweight authentication proxy could enable SDR to integrate with GSM using existing Dapr capabilities.
- **K3s Deployment:** Technically feasible but discouraged due to operational complexity and limited support ownership.

## Next Steps
1. **Atlan:** Validate local secret-file refresh and assess GSM proxy feasibility.
2. **Internal Team:** Confirm security acceptability of local secrets and AWS access restrictions.
3. **Kailash:** Review viable options and provide architectural direction based on security, effort, and timelines.

## Conclusion
No final solution was selected. Local file-based secrets and GSM proxy integration are the leading near-term options, with AWS deployment remaining the preferred long-term strategy.

**Immediate Priority:** Complete technical validation, obtain security approval, and secure Kailash's decision on the implementation path.