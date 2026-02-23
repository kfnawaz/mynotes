# Tableau, Snowflake & 10443 Issue Summary

---

## 1️⃣ Tableau Setup – Status & Resolution

### What Happened
- Initial setup was blocked due to missing JWT-related information from the JPM side.
- One required value (username tied to JWT/FID) was not provided.
- The team reverse-engineered the configuration:
  - Identified possible FIDs
  - Mapped associated email addresses
  - Guessed the correct username successfully

### Current Status
- Tableau connector is now progressing.
- Production workflows are running successfully.
- Issue was not technical — it was an access/credential completeness issue.

### Key Insight
This was a configuration transparency issue, not a platform limitation. Once credentials were correctly inferred, it worked.

---

## 2️⃣ Snowflake Setup – JDBC Communication Failure

### Observations
- Snowflake connector uses **JDBC**.
- Connection works:
  - ✅ From the plain VSI box (outside Kubernetes)
- Connection fails:
  - ❌ From inside Kubernetes pod (secure agent)

### Error Seen
- JDBC communication error  
- Socket connection timeout  
- Network timeout behavior  

### Important Contrast
- Databricks works fine because:
  - Uses REST APIs
  - Not dependent on JDBC
- Snowflake relies on:
  - SQL execution
  - JDBC
  - System tables access

### Current Status
- Engineering actively investigating
- Multiple engineers involved
- No ETA yet
- Work in progress

### Leading Theories
- Proxy configuration differences
- Java environment variables inside pod
- Kubernetes environment-level settings
- Port translation behavior
- Network component interfering

This is clearly not a simple credential issue — it’s a network/JDBC execution path problem.

---

## 3️⃣ The 10443 Port Issue – DR Environment Only

### What’s Happening
- UI configuration shows **port 443 only**
- Yet traffic is observed hitting **10443**
- This behavior:
  - ❌ Happens only on DR server
  - ✅ Does NOT happen on Prod server
- EPV/IAM authentication service runs only on Prod
- DR workflows use Prod EPV

So EPV is not the root cause.

---

## Key Question Raised

**Where is port translation happening?**

Because:
- Workflow config specifies 443
- Code does not specify 10443
- Env files do not contain 10443
- No 10443 in config directories
- Not visible in UI configuration

So where does 443 become 10443?

---

## Possible Translation Points Identified

Port translation could occur in:
- Kubernetes ConfigMaps
- Pod definitions
- Ingress rules
- Proxy layer
- Custom container configuration

> Note: Port translation would NOT appear in the local file system if defined in Docker image or Kubernetes layer.

---

## Strong Hypothesis

JPMC has multiple proxies:
- 10443
- 11443

Likely scenario:
- Proxy performs port translation
- Based on source box or routing rule
- DR traffic is routed differently than Prod
- One proxy may be whitelisting ThoughtSpot traffic differently

This would explain:
- Why Prod works
- Why DR fails
- Why port shows 10443 only in DR
- Why config files don’t show it

---

## Risk / Operational Concern

Running the failing workflow on DR:
- Triggers alerts due to 10443 traffic
- Alerts go up to CIO monitoring
- Requires justification

Team considering:
- Capturing full pod logs during runtime
- Capturing pod descriptions
- Comparing UAT vs DR configuration
- Investigating ConfigMaps

---

## Executive-Level Summary

| Area        | Status            | Root Cause Type                   | Confidence |
|-------------|-------------------|-----------------------------------|------------|
| Tableau     | Working           | Credential/JWT misalignment       | Resolved   |
| Snowflake   | Failing           | JDBC network/proxy issue          | Investigating |
| 10443 (DR)  | Failing (DR only) | Likely proxy/K8s port translation | High Suspicion |

---

## Bottom Line

- **Tableau**: Solved through credential discovery.
- **Snowflake**: Failing due to JDBC-level communication timeout from inside Kubernetes.
- **10443 issue**: Likely environment-specific port translation happening at proxy or Kubernetes layer in DR only.