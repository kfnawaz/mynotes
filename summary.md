# PCI Workspace Metadata Crawling Discussion Summary

## Context

The team needs to crawl metadata from the **CCP Risk PCI Databricks workspace** using Data Compass metadata extractors.

Current challenge:

- The metadata extractor runs in a **non-PCI environment**.
- It requires **read-only access** to a PCI-designated workspace.
- Such access is currently restricted due to PCI compliance controls.

---

## Problem Statement

The Data Compass extractor needs to access metadata from a PCI workspace, but:

- The extractor is hosted in a non-PCI environment.
- Cross-boundary access from non-PCI to PCI environments is not automatically permitted.
- The team is seeking a compliant path to enable metadata extraction.

---

## Key Findings

### No Technical Workaround

There is currently **no approved technical workaround** to bypass the PCI restrictions.

The recommended approach is to pursue a formal exception process.

### CTC Exception Required

The team should engage the **CTC (Cyber Technology Controls)** team and request an exception approval.

The exception request must demonstrate:

1. The extractor accesses **metadata only**.
2. The APIs being used **cannot access or expose actual data**.
3. PCI-sensitive data is never read, copied, or transmitted.

---

## Evidence Required for Approval

The team should prepare documentation proving:

### Metadata-Only Access

- Data Compass only extracts metadata.
- No table contents are read.
- No row-level data is accessed.
- No PCI data is retrieved.

### API Behavior

Provide evidence that the APIs used:

- Return metadata only.
- Do not expose underlying data.
- Cannot be used to retrieve PCI-sensitive information.

---

## Existing Precedent

A similar exception was reportedly approved previously for **HCD workspaces**.

As a result:

- Data Compass currently has access to HCD environments.
- Access was granted through an approved exception process.
- The PCI request is expected to follow a similar path.

---

## Current Blocker

The team needs:

- Workspace onboarding
- Warehouse creation
- **CATS (Cross Account Data Sharing)** access

The inability to obtain the required CATS access is the immediate blocker preventing progress.

---

## Recommended Next Steps

### 1. Initiate CTC Exception Process

- Contact the CTC team.
- Explain the metadata-only use case.
- Request an exception for non-PCI to PCI metadata access.

### 2. Prepare Supporting Evidence

Document:

- Metadata extraction architecture.
- APIs used by the crawler.
- Proof that data access is not possible.
- Security controls and limitations.

### 3. Engage Daniel ("Dan")

Dan was identified as the likely point of contact for these approvals.

Actions:

- Obtain an introduction.
- Share architecture and use case details.
- Walk through the exception request.

### 4. Schedule Follow-Up Discussion

A follow-up meeting can be arranged with:

- Dan
- Data Compass representatives
- Relevant CTC stakeholders

The speaker offered to:

- Provide an introduction.
- Share historical context.
- Participate in future discussions if needed.

---

## Outcome

### Agreed Path Forward

Pursue a **CTC exception approval** similar to the previously approved HCD exception.

Success depends on demonstrating:

- Metadata-only access.
- No PCI data exposure.
- Read-only access patterns.
- Compliance with existing security controls.

Once approved, the team should be able to:

- Obtain the required CATS access.
- Complete PCI workspace onboarding.
- Create the necessary warehouse resources.
- Enable Data Compass metadata crawling for the PCI workspace.