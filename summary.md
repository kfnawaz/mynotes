# 🔑 Discussion Summary: Atlan + Snowflake Integration Constraints

---

## 🧩 Key Discussion Points

### 1. Core Problem
- Atlan requires access to:
  - Snowflake **metadata (account usage)**
  - **Query history** for lineage reconstruction
- Query history may contain **sensitive / PII data**
- Unsanitized exposure is **not allowed**

---

### 2. Existing Baseline Approach
- A **sanitized Snowflake account usage layer** already exists
- DataHub approach:
  - Does **not use query history**
  - Uses **sanitized metadata only**
  - Still provides **usable lineage (non-ideal but functional)**

---

### 3. Key Constraints

#### 🔐 Security Constraints
- No access to **unsanitized query history**
- No reliance on **vendor-side masking**
- All data must be **pre-sanitized**

#### ⚙️ Atlan Product Limitations
- Cannot:
  - Disable query history ingestion
  - Selectively pull from tables/views
- Can only:
  - Switch **entire schema (not partial sources)**

#### 🏗️ Infrastructure Constraints
- Team will **NOT clone or duplicate full Snowflake account usage**
  - High cost and overhead
  - Not justified for non-primary catalog

---

## ⚖️ Options Evaluated

| Option | Status | Reason |
|------|--------|--------|
| Expose raw Snowflake data | ❌ Rejected | Security risk (PII exposure) |
| Let Atlan mask data internally | ❌ Rejected | Data still exposed pre-masking |
| Clone full sanitized schema | ❌ Rejected | High overhead, not strategic |
| Use sanitized views only | ⚠️ Limited | Blocked by Atlan limitations |
| Provide query history in separate schema | ⚠️ TBD | Needs vendor validation |
| Disable query history usage | ⚠️ Preferred | Depends on Atlan capability |
| Follow DataHub model | ✅ Recommended | Proven workaround |

---

## 🧠 Strategic Insight
- This is a **product limitation + governance issue**, not just technical
- Organization stance:
  - ❌ No infra duplication
  - ❌ No security compromise
- Atlan lacks flexibility compared to DataHub

---

## 🚀 Next Steps

1. **Engage Atlan Vendor**
   - Can query history ingestion be disabled?
   - Can extractor be customized (table-level control)?
   - Can query history be isolated in a separate schema?

2. **Validate Partial Workaround**
   - Test if isolated query history schema works

3. **Explore DataHub Approach**
   - Connect with DataHub team (Ram Gupta)
   - Understand how lineage works without query history

4. **Assess Alternative Integration**
   - Evaluate using DataHub design patterns (not data reuse)

---

## 📌 Action Items

### 👤 Team (Nawaz / Engineering)
- Share DataHub contact (Ram Gupta)
- Coordinate with Atlan vendor
- Document Atlan limitations clearly

---

### 🏢 Atlan Vendor
- Clarify:
  - Query history toggle capability
  - Extractor customization support
  - Schema vs table-level flexibility

---

### 🛡️ Architecture / Security
- Reconfirm:
  - No unsanitized data exposure
  - No approval for schema duplication

---

### ⚙️ Engineering
- Evaluate:
  - Feasibility of isolated query history schema
  - Lightweight alternatives to full schema cloning

---

## ⚠️ Bottom Line

- Security stance: **Non-negotiable**
- Infra stance: **No duplication**
- Limitation: **Atlan product capability**

👉 Path forward:
- Either proceed **without query history (DataHub model)**
- Or accept **limited functionality unless Atlan adapts**