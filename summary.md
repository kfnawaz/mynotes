# Meeting Minutes Summary

## Purpose
Discuss options for enabling Atlan to access Tableau Cloud metadata and clarify next steps for onboarding the Atlan FID.

## Key Discussion Points

### Metadata Access Options
- Two viable approaches:
  1. **Connected App** using a JSON Web Token (JWT)
  2. **FID-based access** onboarded to Tableau Cloud, using APIs/VDS to pull metadata (e.g., Admin Insights)
- Both approaches are supported by documentation and considered acceptable.

### Preferred Data Source
- **Admin Insights** is recommended as it contains comprehensive metadata.
- Internal teams are already pulling Admin Insights data into Databricks; Atlan is attempting to connect in parallel.

### Roles & Permissions
- **Explorer role** is sufficient; roles are not a major concern.
- Primary focus is correctly onboarding the FID and enabling API access.

### FID Onboarding Requirements
- **Critical prerequisite:** The FID must have a dedicated email address.
- Tableau Cloud requires every user/FID to have an email and license.
- No ticket progress is possible until the email exists.

### Tickets & Process
- Existing tickets are correct but blocked due to the missing FID email.
- Required actions:
  - Create the external email for the FID.
  - Add a comment to tickets noting the email is in progress.
  - Once created, add the FID to the appropriate dev/pre-prod group (and later an Atlan FID group in prod).

### Pilot Scope
- Initial pilot will start with **Finance** (CCB and CCB Build).
- Long-term expectation is expansion to all corporate technology platforms.

### Connected App Clarifications
- Connected apps are **site-specific**.
- Separate credentials exist for build vs. prod; a mismatch likely caused a recent portal issue.
- Resetting secrets/client IDs can break integrations and must be coordinated.

## Action Items
1. **Navez**: Create the external email address for the Atlan FID (priority).
2. **Navez**: Update all related tickets with a comment about the email creation status.
3. **Tim**: Review tickets after the email is created and handle group assignments.
4. Proceed with **CCB / CCB Build** pilot once FID onboarding is complete.

## Conclusion
Both connection approaches are valid, but progress is blocked until the FID email is created. Once completed, onboarding and pilot execution can move forward quickly.
# Meeting Summary

## Purpose
Align on how Atlan will access Tableau Cloud metadata and unblock FID onboarding.

## Key Points
- **Access Options**
  - Connected App (JWT-based)
  - FID-based API access (preferred for Admin Insights)
- **Data Source**
  - Admin Insights recommended (complete metadata set).
- **Roles**
  - Explorer role is sufficient; roles are not a blocker.
- **Blocker**
  - FID **must have an email address** (Tableau Cloud requirement).  
  - No ticket progress possible until this is done.
- **Tickets**
  - Tickets are valid but stalled due to missing FID email.
  - Add comments noting email creation in progress.
- **Pilot Scope**
  - Start with **Finance (CCB / CCB Build)**; expand later.
- **Connected App Notes**
  - Site-specific credentials (build vs prod).
  - Credential mismatch likely caused recent portal issue.

## Action Items
1. **Navez**: Create FID email (urgent).
2. **Navez**: Update tickets with status comment.
3. **Tim**: Review tickets and add FID to required groups after email creation.

## Conclusion
Both access models work, but FID email creation is the critical prerequisite to proceed.
