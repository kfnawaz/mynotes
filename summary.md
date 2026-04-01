# 📝 Meeting Minutes — Tableau Lineage Discussion

## Objective
Clarify Tableau lineage capabilities and address gaps, especially for file-based sources (Hyper files).

---

## Key Points Discussed

- Lineage is currently supported for:
  - Snowflake  
  - Databricks  
- Lineage works **only for direct connections** from Tableau to these sources.
- Lineage is **auto-generated from metadata** (no separate mining required).

- **Major Limitation:**
  - File-based sources (CSV, Excel, Hyper files) are **not captured in lineage**.
  - Hyper file support is **unclear and currently missing**.

- **Current Reality:**
  - ~95% of dashboards rely on Hyper files.
  - Direct connections represent only ~10–20% of use cases.

- Result:
  - Lineage coverage is **limited and incomplete** for real-world usage.

---

## Proposed Approaches

- Explore OpenLineage for partial lineage via file tracking.
- Investigate feasibility of enhancing connectors to support Hyper files.
- Consider leveraging ETL tools (e.g., Alteryx) for upstream lineage.

---

## Decisions / Agreements

- No immediate solution committed.
- Need to validate:
  - Hyper file lineage feasibility
  - Real-world usage patterns (file vs direct connections)

---

## Action Items

- **Engineering (Avinash):**
  - Investigate Hyper file structure and lineage feasibility
  - Explore OpenLineage-based workaround

- **Team:**
  - Provide sample dashboards (direct + file-based)
  - Validate % distribution of dashboard types

- **Product Team:**
  - Review roadmap and feasibility
  - Provide direction on potential enhancements

---

## Next Steps

- Reconvene in a few days with:
  - Feasibility assessment
  - Initial findings / possible approaches

- No ETA committed yet; direction expected first.

---

## Key Risk

- Without Hyper file support, Tableau lineage will **not meet user expectations** or provide full value.