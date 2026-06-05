# Databricks App Space / Workspace Offboarding Discussion

## Background

- The team originally onboarded a Databricks workspace/app space using an initial naming convention.
- Later, separate **HCD** and **Non-HCD** app spaces were created using the correct naming standards.
- The original app space is now obsolete and needs to be removed.
- A newly created role was mistakenly onboarded into the old app space instead of the correct Non-HCD app space.

---

## Key Findings

### 1. Managed Instances Cannot Be Modified Directly

- The app spaces shown in MTH are managed entities.
- They cannot be manually moved or edited within Entitlement Manager.
- Visibility can be controlled by marking them as requestable or non-requestable, but that does not remove the underlying workspace.

### 2. Custom Role Requires Formal Removal Process

- The incorrectly created role appears to be a Databricks custom role.
- Custom roles must be deleted through the Databricks self-service/Core API process.
- If deletion is not available through self-service, Databricks Engineering will assist.

### 3. Obsolete App Space Must Be Offboarded

- The primary objective is to remove the original app space/workspace onboarding.
- This requires an official intake request.
- The request must specify:
  - Workspace ID of the app space to be removed.
  - Exact app space (SEAL use case) name to be deleted.
  - The other valid app spaces that must be retained.

### 4. No Catalogs Exist Yet

- The onboarding is still in its early stages.
- No Databricks catalogs have been created under the workspace.
- This simplifies the offboarding process.

---

## Expected Offboarding Process

1. Submit an intake request with:
   - Workspace ID.
   - App space/SEAL use case to remove.
   - Other app spaces that should remain.

2. Databricks team validates the request.

3. Core API workflow generates AO/IO approvals.

4. Once approved:
   - MTH synchronization removes the app space.
   - Associated groups are deleted.
   - The mistakenly created custom role should also be removed automatically if it belongs to that app space.

5. The role can then be recreated correctly in the intended Non-HCD app space.

---

## Action Items

### Nawaz

- Identify and provide the exact Workspace ID for the obsolete app space.
- Submit an intake request requesting:
  - Removal of the obsolete app space.
  - Retention of the two valid app spaces.
- Include all relevant app space names to avoid accidental deletion.

### Databricks Team

- Validate the workspace and app space mappings.
- Confirm the correct offboarding scope.
- Execute the offboarding process after approvals.
- Verify that the associated custom role is removed as part of the cleanup.

---

## Outcome

The agreed approach is to **offboard the original obsolete Databricks app space through the formal intake process**, which should also remove the incorrectly created role. After cleanup, the role can be recreated in the correct Non-HCD app space.