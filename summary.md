# Meeting Summary

## 1. k3s Startup Issue

The team found that the long k3s startup time is likely caused by the **k3s SQLite/Kine state database** growing too large.

### Key Findings

- k3s is using **SQLite/Kine** instead of an external etcd database.
- The k3s state database has grown to approximately **232 GB**.
- The database contains around **13.3 million object revision records**.
- These revision records track changes to Kubernetes objects such as:
  - Pods
  - Secrets
  - Jobs
  - Cron jobs
  - Custom resources
- A cleanup, pruning, or vacuum process may not be running correctly.
- Slow SQL messages were observed, indicating that database operations are taking too long.
- The large state database is likely contributing to the long startup time and instability.

---

## 2. Recovery Options Discussed

### Option 1: Move the Large State DB to Backup

This option involves stopping k3s, moving the large database files to a backup location, and allowing k3s to recreate a fresh state database.

#### Steps

1. Stop k3s.
2. Move `state.db` and related files to a safe backup location.
3. Restart k3s.
4. Allow k3s to recreate a new state database.
5. Recreate any missing secrets if required.

#### Risk

- Some manually created secrets may not be restored automatically.
- The team may need to recreate secrets for each namespace.

---

### Option 2: Vacuum / Clean Up the Existing Database

This option involves running a cleanup or vacuum operation on the existing database.

#### Considerations

- This is a more surgical approach.
- The team was hesitant because `sqlite3` was not installed.
- The database is already very large, making this option riskier and potentially slower.

---

### Option 3: Reinstall / Redeploy

This option involves reinstalling or redeploying the environment.

#### Considerations

- A redeploy may not resolve the issue if the same large state database is reused.
- Some separate components, such as the token generator jobs, may need to be redeployed separately.
- The standard secure agent redeploy process may not recreate all manually created secrets.

---

## 3. Production Snowflake / LRI Workflow Update

The team also discussed the production Snowflake workflow issue.

### Key Findings

- The LRI workflow issue was partly resolved.
- Owen made an **Atlan-side workflow template update**.
- After the template update, the workflow reached the expected **run-on-agent** path.
- The workflow is now failing due to a **Snowflake warehouse permission issue**.
- The likely fix is to grant the role permission to use and operate the configured Snowflake warehouse.

### Important Note

Future new Snowflake workflows may hit the same Atlan-side template issue until engineering provides a permanent fix.

---

## 4. Decision Made

The team decided not to wait for k3s to recover on its own.

### Agreed Approach

The team chose to:

1. Stop k3s.
2. Move the large `state.db` and related files to a backup location.
3. Restart k3s.
4. Let k3s recreate a smaller fresh database.
5. Recreate any missing secrets if needed.

---

## 5. Current Status

After moving the database:

- A new database was created.
- The new database appeared significantly smaller.
- k3s started rebuilding resources.
- The team continued monitoring the rebuild and recovery process.

---

## Overall Conclusion

The UAT issue appears to be caused by a **bloated k3s SQLite/Kine state database**, likely due to object revision records not being cleaned up correctly.

The production LRI Snowflake workflow issue has moved past the earlier workflow-template routing problem and is now blocked by a **Snowflake warehouse permission issue**.