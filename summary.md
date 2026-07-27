# Smart Approvals Integration Meeting Summary

## Integration Approach

Smart Approvals communicates back to the requesting application through a **postback (listener) endpoint**. Two implementation options were discussed:

### Option 1: Apigee Proxy

- Create a proxy endpoint in Apigee.
- Can be reused by multiple applications.
- Secured using an authentication token in the request header.
- Centralized API management.

### Option 2: Application-Hosted Endpoint

- Expose a listener endpoint directly from the application (e.g., GAAP).
- Provides greater flexibility and control over implementation.
- Allows the application team to choose its preferred technology stack and security model.

**Recommendation:** Since the application is hosted in GAAP, an application-hosted endpoint appears to be the most appropriate approach.

---

# Smart Approvals Postback Flow

After an approval request is submitted:

1. Smart Approvals sends a confirmation that the request was received.
2. It enriches the request with additional information (for example, manager details retrieved from HR).
3. The application updates its UI or internal state using this information.
4. Whenever the approval status changes, Smart Approvals sends another notification to the listener endpoint.

The payload includes:

- Smart Approvals request identifier (internal unique ID)
- Current approval status
- Approval updates
- Rejection information
- Cancellation information (multiple cancellation reasons supported)

Supported statuses include:

- Pending
- Approved
- Rejected
- Cancelled

The application listener must be able to process every supported payload type.

---

# Listener Response Requirements

After receiving a callback, the application should return:

## Success Response

Return a success response when:

- The payload was received successfully.
- The payload was understood.
- The application can process it.

## Error Response

Return an error response when:

- The payload is invalid.
- Required data is missing.
- The payload cannot be processed.

---

# Authentication

Authentication depends on how the application secures its APIs.

Possible scenarios include:

- ADFS authentication
- Domain group membership
- FID-based authorization
- API entitlements

If the application protects its APIs:

- Smart Approvals' FID must be added to the required domain group.
- Appropriate entitlements may need to be created (via MyTechHub).
- Smart Approvals will then be authorized to invoke the listener endpoint.

If the endpoint does not require those controls, no additional provisioning is necessary.

---

# Ownership of Data Changes

An important clarification was made:

Smart Approvals **does not modify the application's data**.

Instead, Smart Approvals sends notifications such as:

> "This request has been approved."

The receiving application is responsible for:

- Recording the approval
- Updating its own database
- Refreshing the UI
- Performing any downstream business logic

---

# Retry Behavior

If the listener endpoint is unavailable:

### UAT

- Retry up to **10 times**

### Production

- Retry up to **20 times**

Retry interval:

- Approximately **every 15 minutes**

---

# Request Status API

In addition to postbacks, Smart Approvals exposes a **Request Status API**.

The application can query the latest approval status using the Smart Approvals request ID.

This provides a recovery mechanism if:

- Postbacks are missed
- Listener failures occur
- Status synchronization is required

---

# Workflow Capabilities

Smart Approvals supports highly configurable workflows.

Supported capabilities include:

- Unlimited approval steps
- Sequential approvals
- Parallel approvals
- Individual approvers
- Group (pool) approvers
- Multiple approval roles/personas
- Delegate restrictions
- Dynamic workflows per request

Each approval request can define its own workflow.

There is **no requirement** for a fixed workflow template.

---

# Group-Based Approvals

For pooled approvals, the request payload should specify:

- Group name
- Members (SIDs/FIDs)
- Approval sequence
- Whether:
  - Any one member may approve
  - All members must approve

---

# Dynamic Approvers

Smart Approvals can determine approvers dynamically.

Example:

If the workflow specifies:

> Requestor's Line Manager

Smart Approvals retrieves the manager information directly from HR and routes the approval accordingly.

---

# Payload Modeling Discussion

Rather than trying to understand every supported payload scenario individually, the team agreed on a simpler approach:

1. Select one real workflow from the application.
2. Model the Smart Approvals payload for that workflow.
3. Validate the payload with the Smart Approvals team.
4. Expand to additional workflow variations later.

This will provide an end-to-end validated example before handling edge cases.

---

# Action Items

## Application Team

- Build the listener (postback) endpoint.
- Share endpoint details with Smart Approvals.
- Model one complete approval workflow payload.
- Include:
  - Approval steps
  - Groups
  - Approvers
  - Sequence
  - Approval rules
- Verify authentication requirements.
- Confirm any required FID/domain-group entitlements.

## Smart Approvals Team

- Configure communication with the application's listener endpoint.
- Review the modeled payload.
- Validate the workflow implementation.
- Provide development support during testing.

---

# Next Steps

1. Build and expose the listener endpoint.
2. Share endpoint details with Smart Approvals.
3. Configure two-way communication.
4. Create a complete sample payload for one business workflow.
5. Review the payload jointly with the Smart Approvals team.
6. Execute end-to-end testing.
7. Expand support to additional workflow configurations after the initial flow is validated.

---

# Follow-up Meeting

A follow-up session was tentatively scheduled for:

- **Time:** 11:00 AM Eastern
- Smart Approvals development team members may join to review the payload and provide implementation guidance.