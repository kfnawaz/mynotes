Here’s the **latest update on the Tableau workflow**, based strictly on the transcript.

---

## 1️⃣ Tableau Ingestion – Current Failure

### 🔎 Root Cause Identified

> **“etcd server request is too large”**

- Extracted metadata file size: **1.6 MB**
- Current system limit: **~1.5 MB**
- So it’s failing because it’s **slightly over the request size limit**

This is happening during the **AWS polling/ingestion step** after extraction completes successfully.

Important:
- The extraction itself succeeds.
- Failure occurs when pushing the metadata payload back.

---

## 2️⃣ Incremental Extraction Status

There is **no incremental extraction for Tableau** currently.

That means:
- Every run is a **full extract**
- It may take long each time
- Large metadata payloads are expected

This is a design limitation right now.

---

## 3️⃣ Immediate Mitigation Strategy

Two approaches discussed:

### Option A (Quickest)

Reduce scope temporarily:
- Instead of pulling 54 projects
- Test with 1–2 projects only
- Validate end-to-end success

This is a **temporary workaround** just to confirm pipeline behavior.

---

### Option B (Proper Fix)

Increase the request size limit on the polling step.

Open questions:
- Is this limit **generic for all connectors**, or Tableau-specific?
- If generic, it could affect future large Databricks ingestions too.

Engineering is investigating where the limit is set.

---

## 4️⃣ Performance Optimization Applied

They enabled a **beta feature**:

> “Enable Source Level Filtering”

Purpose:
- Improves filtering efficiency
- Reduces extraction time
- Prevents scanning everything before filtering

However:
- This improves performance
- It does NOT fix the request-size failure

Still useful for future runs.

---

## 5️⃣ Image Version Check

There was concern that this might again be an **image/version issue** (like previous Snowflake OAuth problems).

They verified in UAT:

- Connector image: **12.16**
- Extractor image: **1.0.5**
- Argo Exec: **3.4.14**

Conclusion:
- Images appear up-to-date.
- Not currently believed to be an image mismatch issue.

---

## 6️⃣ Current Status

- Small test run initiated (2 projects)
- Engineering reviewing request size limit
- Snowflake run was also triggered separately (likely image fix applied)
- Waiting for confirmation on increasing payload limit

---

# 🔵 Bottom Line

The Tableau workflow issue is **not a connector logic problem**.

It is:

> A metadata payload size limit issue during ingestion.

Next decisive action:
- Increase request size limit
- Or reduce ingestion scope temporarily

If I were advising execution:
Don’t keep slicing projects manually. Fix the limit properly. Tableau metadata can grow — this will keep coming back otherwise.