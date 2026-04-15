# Metadata Change Approval & Sync Workflow — Use Case

## Objective
Enable a structured approval workflow for **metadata changes in Data Compass**, ensuring governance, auditability, and synchronization with **Fusion**.

---

## Core Problem
- Metadata updates (e.g., reports, designations, classifications) require controlled approvals.
- Users should not be forced to approve changes داخل Data Compass UI.
- System must:
  - Capture **who changed what**
  - Capture **who approved/rejected**
  - Maintain a **complete audit trail**
  - Sync approved changes to **Fusion**

---

## Proposed Solution

### 1. Approval Workflow Integration
- Metadata changes in **Data Compass UI** trigger approval workflows.
- Approval requests are routed via **Smart Approval system**.
- Routing is based on **roles and domain (e.g., Finance CDL)**.

---

### 2. Approval Handling
- Approvers:
  - Receive requests in **Smart Approval queue**
  - Can **approve/reject externally**
- No need to log into Data Compass for approvals.

---

### 3. Audit Trail Capture
- Data Compass captures:
  - Change event
  - Approver identity
  - Timestamp
- Maintains **full audit trail for governance and compliance**

---

### 4. Provisioning & Sync
- After approval:
  - Changes are committed in **Data Compass**
  - Approval metadata is stored
  - Data is synced to **Fusion**, including audit details

---

## Scope Clarification (Key Decision)

### Option A: Approval Only (Preferred)
- Smart Approval handles **approval routing only**
- Data Compass handles:
  - State changes
  - Audit trail
  - Sync to Fusion

---

### Option B: Approval + Provisioning
- Smart Approval also handles provisioning
- Adds complexity and tighter coupling

---

## Workflow Types

### Primary (In Scope)
- **Quality Control (QC) Workflows**
  - Triggered on every metadata change
  - Requires approval before persistence

---

### Secondary (Partially / Out of Scope)
- **Annual Certification / Recertification**
  - Periodic validation workflows
  - May not require Smart Approval integration

---

## Access & Role Model
- Roles are defined in **Data Compass**
- Roles determine:
  - Who can modify metadata
  - Who receives approval requests
- No changes to roles in external systems (Atlan / Fusion)

---

## Key Design Principles
- Decouple **Data Compass UI** from approval system
- Use Smart Approval only for:
  - Routing
  - Decision capture (approve/reject)
- Keep:
  - Change context
  - Audit trail
  within Data Compass
- Ensure Fusion receives **approved and audited data only**

---

## Open Questions
- Should we use **Smart Approval vs Smart Flow**?
- Where should provisioning responsibility lie?
- Final workflow definitions for stakeholder alignment

---

## Summary
A governed metadata workflow where:
- Users update metadata in Data Compass  
- Approvals are handled externally via Smart Approval  
- Data Compass remains the **system of record**  
- Approved changes are synced to Fusion with full audit trace  