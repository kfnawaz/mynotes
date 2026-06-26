# Data Compass Crawling / Strategic Browse Access Summary

## Overview

The discussion focused on how **Data Compass / Atlan** is currently crawling assets from the **CCB Risk HCD** and **PCI** workspaces, and what needs to happen before enabling the new **strategic crawling / browse-role access** approach. :contentReference[oaicite:0]{index=0}

---

## Current Setup

There are currently two direct crawler connections:

| Connection / Workspace | Approx. Database Count | Current Status | Lineage Mining |
|---|---:|---|---|
| CBR Prod / HCD | 34 databases | Crawling enabled | Not enabled |
| CBR Prod PCI | 17 databases | Crawling enabled | Not enabled |

These connections are already crawling assets into Data Compass / Atlan.

---

## Direct Browse Access Assets

For assets currently coming directly from the **CBR HCD** and **CBR PCI** connections:

- No rollback is needed.
- No major impact is expected.
- The assets remain the same.
- The catalogs, schemas, and table names remain the same.
- The connection/channel remains the same.
- Only the **authorization method** changes.

In other words, the FID will move to the new strategic browse-role access model, but the asset identity should remain stable.

---

## Catalog Share Assets

A separate issue exists for assets coming through **catalog sharing via Odessa Gold Prod**.

Some CCB Risk catalogs are currently shared into Odessa Gold. Because of that, Atlan treats those assets as belonging to the **Odessa Gold connection/channel**, even though the original source is CBR Prod.

When the same assets are later discovered directly through **CBR Prod** using the new strategic browse-role access, Atlan may treat them as **different assets**, because the discovery channel changes.

---

## Main Risk

The main risk is with **product mappings**.

Some assets coming through catalog share may already be mapped to products. If catalog sharing is disabled, those shared assets may disappear from Atlan. Then, when the same assets are rediscovered through the strategic CBR Prod crawler, they may appear as new assets and need to be remapped.

---

## Impact Assessment

| Asset Source | Expected Impact | Reason |
|---|---|---|
| Direct CBR HCD crawl | No major impact | Same connection/channel; only authorization changes |
| Direct CBR PCI crawl | No major impact | Same connection/channel; only authorization changes |
| Odessa Gold catalog-share assets | Impact expected | Channel changes from Odessa Gold to direct CBR Prod |
| Product mappings on shared assets | Impact expected | Existing mappings may need to be moved to newly discovered strategic assets |

---

## Agreed Understanding

- **CBR HCD and PCI direct crawled assets should not be impacted.**
- **Assets coming through Odessa Gold catalog share will be impacted.**
- Catalog sharing will need to be disabled.
- Once catalog sharing is disabled, shared assets may be removed from Atlan.
- The new strategic crawler will rediscover those assets directly from CBR Prod.
- Product mappings will need to be reviewed and remapped to the newly discovered strategic assets.

---

## Required Coordination

This change requires coordination across multiple teams because product mappings may be affected.

The strategic crawling change should be delayed by **one to two weeks** to allow time for planning, especially because the July 4th holiday week may slow coordination.

---

## Action Items

### 1. Create a tracking table

Create a table with the following fields:

| Field | Purpose |
|---|---|
| Database name | Identify the database being crawled |
| Workspace/source | Identify whether it belongs to HCD, PCI, CBR Prod, etc. |
| Connection/channel | Identify whether it comes from CBR, PCI, or Odessa Gold |
| Discovery method | Direct browse access or catalog share |
| Product mapping status | Whether the asset is mapped to any products |
| Expected impact | No impact / remapping required |
| Remediation action | What needs to be done before or after strategic crawling |

---

### 2. Send an email summary

Send an email covering:

- Current crawling setup
- Difference between direct crawl and catalog-share crawl
- Which assets are not expected to be impacted
- Which assets are expected to be impacted
- Why catalog-share assets may be duplicated or remapped
- Required coordination steps
- Product mapping remediation plan
- Proposed delay of the strategic crawling change

---

### 3. Include the right stakeholders

The email should include:

- PMs
- Product technology teams
- Data Compass / Atlan stakeholders
- CCB Risk product owners
- Teams responsible for catalog sharing and product mappings

---

### 4. Continue access onboarding

Proceed with onboarding and provisioning access to the HCD and PCI workspaces.

However, do **not** enable the new strategic crawler until the coordination and remediation plan is agreed.