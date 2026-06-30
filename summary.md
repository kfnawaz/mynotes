# Meeting Summary: Atlan AWS IAM Role & Connector Configuration

## Objective

Finalize the AWS IAM configuration required for the Atlan connector and determine whether the connector should use a **Direct Connector** or **Secure Agent (SDR)** deployment before the project deadline.

---

# Key Discussion Points

## 1. IAM Trust Policy Configuration

- Atlan-side configuration has already been completed.
- The remaining task is to update the **AWS IAM trust policy** by adding the Atlan-provided IAM role ARNs.
- Three role ARNs were shared (Dev, UAT, and Prod).
- Each environment's corresponding IAM role must be updated with the appropriate trust policy.
- The team agreed to begin this configuration immediately.

---

## 2. Connector Configuration

After the IAM trust policies are updated, the team will:

- Configure the Atlan connector.
- Test the connection.

The support ticket will remain open until the connector is fully configured and verified.

---

## 3. Kubernetes Secrets Discussion

A question was raised regarding whether the workflow requires Kubernetes Secrets.

### Current Status

- Bearer tokens can already be retrieved from the identity system using GSM.
- A temporary proof-of-concept script can be created to validate the authentication flow.
- This script will not support scheduled execution but is sufficient for initial validation.
- The complete production implementation can be completed afterward.

---

## 4. Direct Connector vs. Secure Agent (SDR)

This became the primary technical discussion during the meeting.

### Current Uncertainty

Existing Databricks and Snowflake workflows use the **Secure Agent (SDR)** model together with a secret store.

It is currently unclear whether this connector should instead use a **Direct Connector**.

### Direct Connector

- Connects over the public internet.
- Appears not to require the same secret-store configuration.

### Secure Agent (SDR)

- Runs inside the customer's Azure environment.
- Pushes metadata into Atlan.
- Uses IAM Role authentication.
- Appears to require additional secret configuration.

The team believes that since all existing connectors currently operate through the Secure Agent, this connector will likely need to follow the same deployment model.

---

## 5. Outstanding Technical Question

The exact secret requirements for the Secure Agent deployment remain unclear.

The Atlan team will verify with Owen:

- Whether Secure Agent is mandatory.
- Which secret(s) are required.
- Where those secrets should be configured.

---

# Decisions

- ✅ Proceed immediately with IAM trust policy updates.
- ✅ Configure and test the connector once IAM configuration is complete.
- ✅ Keep the support ticket open until end-to-end validation is successful.
- ⏳ Confirm whether the connector should use **Direct Connector** or **Secure Agent (SDR)**.
- ⏳ Confirm the required Kubernetes Secret / Secret Store configuration.

---

# Action Items

| Owner | Action |
|--------|--------|
| Infrastructure / RDI Team | Update AWS IAM trust policies with the provided Atlan IAM Role ARNs for Dev, UAT, and Prod environments. |
| Nawaz's Team | Configure and test the connector after IAM policy updates are completed. |
| Atlan Team | Confirm whether the connector should use Direct Connector or Secure Agent (SDR). |
| Atlan Team | Identify the required Kubernetes Secret / Secret Store configuration and communicate the implementation details. |
| All Teams | Complete validation and close the support ticket only after successful connector testing. |

---

# Open Questions

1. Should this connector use the **Direct Connector** deployment model or the **Secure Agent (SDR)** deployment model?
2. If Secure Agent is required:
   - What Kubernetes Secret(s) must be configured?
   - Where should those secrets be stored?
   - How are they referenced by the workflow?
3. Is there any additional configuration required beyond the IAM Role trust policy?

---

# Next Steps

1. Update the IAM trust policies.
2. Configure the connector.
3. Test the connector.
4. Obtain clarification from Owen regarding Secure Agent and secret management.
5. Complete end-to-end validation.
6. Close the support ticket after successful verification.

---

# Overall Outcome

The meeting concluded with agreement to immediately proceed with the **IAM trust policy configuration**, as this step is required regardless of the final deployment architecture. The only remaining blocker is confirmation of whether the connector should run through the **Secure Agent (SDR)** or the **Direct Connector**, along with clarification of the associated Kubernetes secret management requirements. The team emphasized the urgency of resolving these remaining items due to the month-end delivery deadline.