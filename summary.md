# AWS Glue Authentication Discussion Summary

## Objective

Discuss authentication options for connecting the **Atlan Secure Agent** to **AWS Glue** within the constraints of JPMorgan Chase (JPMC) security policies.

---

# Current Findings

## IAM User Access Keys are Not Supported

- JPMC engineering confirmed that **IAM User Access Key / Secret Key authentication is not supported**.
- A support ticket was created with the JADE engineering team.
- Their recommendation is to use an **IAM Role + AssumeRole (Trust Relationship)** approach.

---

## Role-Based Authentication is Blocked

The Secure Agent is currently deployed on an **on-premises Unix VM**.

Because the VM is **outside AWS**, it:

- Cannot be assigned an AWS IAM Role.
- Cannot perform an IAM `AssumeRole` operation directly.

AWS role assumption only works from AWS resources such as:

- EC2
- EKS
- ECS
- AWS Lambda
- Other AWS-managed compute services

---

# Authentication Options Reviewed

## Option 1 — IAM User Access Keys

### Description

Store an AWS Access Key and Secret Key inside the Secure Agent.

### Status

❌ **Rejected**

### Reason

- Violates JPMC security policy.
- Officially not recommended by the JADE engineering team.

---

## Option 2 — Secure Agent (Self-Hosted Runtime)

### Description

- Secure Agent runs on an on-premises VM.
- The VM authenticates to AWS Glue.
- Metadata is collected locally and uploaded to Atlan.

### Requirement

Requires AWS Access Key and Secret Key.

### Status

❌ **Blocked**

### Reason

Since the Secure Agent is running outside AWS:

- It cannot assume an IAM Role.
- The only supported AWS authentication mechanism is Access Keys.
- Access Keys are prohibited by JPMC policy.

---

## Option 3 — Direct Connectivity

### Description

Instead of using the Secure Agent:

- Atlan Cloud directly assumes the Glue Read IAM Role.
- Crawling is performed from Atlan infrastructure.
- Metadata is collected directly by Atlan.

### Status

⚠️ **Technically Possible**

### Limitation

This introduces another architectural concern:

- Metadata extraction occurs from infrastructure outside JPMC.
- Requires architectural and security approval.
- This deployment pattern has not previously been adopted within JPMC.

---

# Root Cause

The discussion concluded that the team is facing a **Catch-22**.

| Constraint | Impact |
|------------|--------|
| Access Keys are prohibited | Secure Agent cannot authenticate |
| IAM Roles require AWS compute | On-prem VM cannot assume roles |
| Secure Agent is hosted on an on-prem VM | Cannot use IAM Roles |

As a result:

> None of the currently available authentication models satisfy all security constraints.

---

# Important Clarifications

The original issue appeared to be related to **IAM Role naming**.

However, during the discussion it became clear that:

- Role naming is **not the blocker**.
- Trust policies are **not the blocker**.
- The actual limitation is the **AWS authentication model** itself.

Because the Secure Agent is not running inside AWS, IAM Roles cannot be used.

This is an AWS architectural limitation rather than an Atlan limitation.

---

# Impact on Future SDR Adoption

The team discussed the upcoming migration from **Secure Agent** to **SDR (Self-Deployed Runtime)**.

### Observation

If SDR is also deployed on an on-prem VM:

- The exact same authentication problem will continue to exist.

The issue would only be resolved if SDR is deployed on AWS infrastructure (such as EC2 or EKS), allowing IAM Role assumption.

---

# Proposed Next Steps

## 1. Document the Current Constraints

Prepare a summary of:

- Available authentication options
- Technical limitations
- Security policy restrictions

---

## 2. Meet with Internal Teams

Schedule a discussion involving:

- JADE Engineering
- Infrastructure Engineering
- Architecture Team
- Atlan

---

## 3. Determine Whether

- A policy exception can be granted
- A new supported authentication model exists
- The runtime should be migrated onto AWS infrastructure

---

# Overall Conclusion

At present, there is **no technically viable authentication solution** for connecting an **on-premises Secure Agent** to **AWS Glue** while remaining compliant with current JPMC security policies.

The issue can only be resolved through one of the following:

1. Deploy the runtime on AWS so IAM Roles can be used.
2. Obtain a policy exception permitting AWS Access Keys.
3. Introduce a new authentication pattern approved by both AWS and JPMC engineering.

Until one of these conditions changes, the Glue integration remains blocked.