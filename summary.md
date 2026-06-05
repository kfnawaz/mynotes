# Meeting Summary – Databricks Workspace Crawling (HCD & PCI)

## Objective
Review progress on enabling DataCompass metadata crawling for Databricks workspaces and discuss a long-term solution for PCI workspaces.

---

# 1. HCD (Non-PCI) Workspace Onboarding

## Current Status
- HCD workspace onboarding is currently in progress.
- The team is onboarding their own newly created HCD workspaces.
- This effort includes CCB Risk HCD environments and other HCD workspaces.

## Clarification
- DataCompass (HCD) can crawl another HCD workspace when the DataCompass FID is granted appropriate access.
- This model is already working for the Odessa Gold HCD workspace.
- The request is to enable the same capability for the CCB Risk HCD workspace.

## Desired Outcome
- Grant the DataCompass FID access to CCB Risk HCD workspaces for metadata crawling.
- Reduce dependency on individual teams manually granting browse access.
- Establish a workspace-level onboarding model similar to Odessa Gold.

## Action Items
- Follow up on the previously opened browse-access ticket.
- Prioritize and complete HCD workspace onboarding activities.
- Evaluate options for broader workspace-level access.

---

# 2. PCI Workspace Crawling Strategy

## Current Situation
- PCI workspaces are currently handled on a case-by-case basis.
- Existing access models provide limited catalog visibility.
- Full crawling capabilities, including lineage extraction, are not currently available.

## Challenges
- Current PCI onboarding does not provide complete DataCompass functionality.
- Similar PCI onboarding requests are expected from other towers.
- The current exception-based model is not scalable.

## Requested Outcomes

### Short-Term
- Enable full metadata crawling capabilities, including lineage, for PCI workspaces where possible.

### Long-Term
- Develop an enterprise-wide PCI onboarding strategy that can be reused across all PCI workspaces.
- Eliminate the need for repeated case-by-case implementations.

## Reference Material
- A reference document was shared with Kailash and Nawaz.
- The document contains potential approaches for PCI workspace access.
- Engagement with the PCI team may be required to determine a viable solution.

## Key Consideration
- CCB Risk is the first PCI onboarding request.
- Similar requests are expected from other organizations, making a strategic solution necessary.

---

# Discussion Outcome

## Nawaz
- Agreed to revisit the PCI onboarding topic.
- Will review the shared reference material.
- Will follow up on the previously opened browse-access ticket.
- Will investigate potential enterprise-wide PCI solutions.

---

# Next Steps

1. Review PCI reference documentation.
2. Investigate scalable PCI onboarding approaches.
3. Follow up on the browse-access ticket.
4. Evaluate workspace-level access options.
5. Prepare recommendations and proposed solutions for review.

---

# Follow-Up Meeting

- Current meeting will be rescheduled.
- Team agreed that **Wednesday next week** is the preferred date.
- Progress updates, findings, and proposed actions will be discussed during the next meeting.