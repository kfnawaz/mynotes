## Meeting Minutes (Brief)

- Two approaches tested:
  - **PCI:** Browse access via Data Compass FID → enabled metadata crawling  
  - **Non-PCI:** Catalog sharing → worked for crawling but failed for lineage mining  

- Decision:
  - De-prioritize **lineage mining**
  - Focus on **asset crawling**

- Key Outcomes:
  - **Browse access approach** confirmed as simpler and preferred (PCI & Non-PCI)
  - Current setup is a **mixed approach** → needs standardization
  - **Catalog sharing to be phased out**
  - Need **direct connection/workflow for Non-PCI**
  - Each catalog requires **explicit browse access grant to FID**
  - “All data browse role” disabled → may require **custom role**

---

## Action Items

### Nawaz
- Create **direct connection/workflow for Non-PCI**
- Configure **catalog filters (if exclusions needed)**

### All Product Teams
- Grant **browse access to Data Compass FID** for each catalog
- Provide **catalog names, workspace details, and request IDs**

### Raja / Sugandi
- Identify **test catalog (lower/UAT environment)**
- Validate **browse-only crawling approach**

### Team (General)
- **Phase out catalog sharing** after validation
- Test in lower environment → **standardize approach across teams**

### Pending
- Create **custom browse role** (since global role disabled)
- Schedule **follow-up meeting next week**