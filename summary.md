# CT Data Compass User Feedback Summary

## What Users Were Looking For

The users represented applications such as **Pulse** and **Frontier Insights** and wanted to understand how to use **CT Data Compass** to find the correct, authoritative, or **“golden” source of data**.

They were specifically trying to determine how to:

- Find data such as **FARM, PPC, MyData**, and other application datasets.
- Identify which published dataset/table is the **official source**.
- Understand the relationship between:
  - Data Products
  - Data Sets
  - Data Distributions
  - Data Services
  - Data Offers
  - Data Contracts
- Determine whether they should search in **CT Data Compass or Fusion**.
- Understand how **Data Offers and Data Contracts** work.
- Understand how to obtain access to a discovered dataset.
- Avoid consuming duplicate, unofficial, or non-authoritative versions of data.

---

## What Was Shown / Explained

The demonstration clarified that **Data Compass primarily catalogs metadata**, rather than directly ingesting business data from systems such as FARM.

If FARM data exists as tables or views in platforms such as **Databricks or Snowflake**, those physical assets can be crawled and represented in Data Compass.

The following concepts were demonstrated or explained:

- A **Data Distribution** represents the physical implementation of data, such as a Databricks or Snowflake table/view.
- A **Data Product** is the business-level representation owned by a product owner and can point to multiple approved tables/distributions.
- FARM-related data was eventually located under a **Vulnerabilities Data Product**.
- That Data Product contained multiple Databricks tables identified by the product owner as the approved sources for consumption.
- An **Approved** Data Product or Distribution should indicate that it is ready for consumption.
- Ownership information can be used to contact the **Data Product Manager/Owner** when the scope or meaning of the data is unclear.
- A **Data Offer** represents a producer's commitment around areas such as:
  - Timeliness
  - Completeness
  - Record counts
  - Data-quality checks
  - Publishing frequency
- A **Data Contract** can then be established between a consumer and producer based on the Data Offer.
- Data Contracts are currently a relatively new capability and have limited adoption.
- Future functionality is planned to integrate Data Compass with the **entitlement/access service**, allowing users to request access directly from the Data Compass UI.
- **CT Data Compass** is intended to be the primary catalog for CT-owned data.
- **Fusion** provides broader firm-wide discovery and can expose CT Data Products that are published from Data Compass.

---

# Feedback and Improvement Opportunities

## 1. Improve Search

The strongest feedback was that searching for a business term such as **“FARM” should directly surface the relevant approved Data Product and its distributions**.

Users should not need to already know:

- CLID/SEAL ID
- Technical table names
- Platform names
- Organizational classifications
- Data Product names

---

## 2. Search Across Metadata and Relationships

Search should work across all related metadata.

For example, if the word **FARM** appears in:

- A table
- A column description
- A Data Set
- A Data Distribution
- A business term
- Lineage
- A Data Product description

then the associated Data Product should appear in the search results.

---

## 3. Clearly Identify the Golden / Authoritative Source

Users need a very obvious indication of which source they are expected to consume.

Possible indicators could include:

- **Authoritative Source**
- **Golden Source**
- **Approved for Consumption**
- **Recommended Source**
- **Official Data Product**

This is especially important when multiple teams publish similar data.

---

## 4. Identify Duplicate or Competing Sources

When multiple teams publish similar data, Data Compass should help users understand:

- Which source is authoritative.
- Which sources are duplicates.
- Which source should be consumed.
- Which sources are deprecated or should not be used.

The system could also warn users when they are viewing a non-authoritative version.

---

## 5. Improve Navigation Between Catalog Objects

The relationship between the following objects was not intuitive:

**Data Product → Data Set → Data Distribution**

Users should be able to easily navigate between these layers.

The application should clearly show how the logical/business representation of data maps to physical tables.

---

## 6. Provide Plain-English Definitions

The distinction between the following concepts was confusing:

- Data Product
- Data Set
- Data Distribution
- Data Service
- Data Offer
- Data Contract

Each concept should have a simple, plain-English explanation directly in the UI.

For example:

> **Data Product:** A business-approved collection of data intended for consumption.

> **Data Distribution:** The physical table or view where that data is available.

---

## 7. Add a Visual Conceptual Model

The application should provide a simple relationship diagram explaining how the objects connect.

For example:

**Business Data Product**
↓
**Data Set**
↓
**Data Distribution**
↓
**Databricks / Snowflake Table**

Additional relationships could show:

**Producer**
↓
**Data Offer**
↓
**Data Contract**
↓
**Consumer**

---

## 8. Improve Business-Language Discovery

Users naturally search using business terminology such as:

- FARM
- PPC
- MyData
- Vulnerabilities
- Audit data

They do not naturally search using:

- Platform names
- Table names
- Application IDs
- CLIDs
- Technical classifications

Search should therefore support:

- Business terms
- Synonyms
- Tags
- Abbreviations
- Semantic search
- Related terms

---

## 9. Explain Why a Search Result Matched

Search results should explain why they were returned.

For example:

> **Vulnerabilities Data Product**

> Matched because: FARM data is contained in 3 associated Data Distributions.

This would make it much easier to understand why a seemingly unrelated Data Product appeared in the results.

---

## 10. Add Direct Access Request Capability

Once a user discovers the correct Databricks or Snowflake table, they should be able to request access directly from Data Compass.

Desired workflow:

**Discover Data**
→
**Review Data Product**
→
**Select Distribution**
→
**Request Access**
→
**Track Approval**

Users should not have to leave Data Compass and independently figure out the entitlement process.

---

## 11. Clarify CT Data Compass vs Fusion

The application and documentation should clearly explain when users should use each platform.

Suggested positioning:

### CT Data Compass

Primary discovery and data-management platform for **CT-owned data and CT users**.

### Fusion

Firm-wide catalog used for discovering data across different Lines of Business.

For CT users, **CT Data Compass should generally be the default starting point**.

---

## 12. Clearly Show Data Scope

Users need to know whether a Data Product contains:

- CT-only data
- A specific Line of Business
- Multiple Lines of Business
- Enterprise-wide data

This information should be visible directly on the Data Product page.

Users should not have to contact the Data Product Owner simply to determine the scope of the data.

---

## 13. Improve Data Contract Visibility and Guidance

Data Contracts are relatively new and currently have limited adoption.

The application should clearly explain:

- What a Data Contract is.
- When a consumer should create one.
- Why it is useful.
- Who creates it.
- How it relates to a Data Offer.
- What happens when a producer does not meet the contract.

---

## 14. Surface Producer Commitments

Producer commitments should be prominently displayed.

Examples include:

- Publishing schedule
- Refresh frequency
- Timeliness
- Record completeness
- Data-quality requirements
- Availability expectations
- SLA/SLO commitments

This would allow consumers to understand the reliability of the data before using it.

---

## 15. Integrate Contracts With Operational Monitoring

Data Compass should eventually show whether producers are actually meeting their Data Contract commitments.

For example:

**Data Contract Status: Healthy**

- Timeliness: Passed
- Record count: Passed
- Data quality: Passed
- Last refresh: On time

Or:

**Data Contract Status: Breached**

- Expected refresh: 10:00 AM
- Actual refresh: 1:35 PM
- Timeliness SLA: Failed

This could be driven by operational events or monitoring platforms such as **Broadsword**.

---

## 16. Improve Onboarding and Help Content

The current help and architecture documentation does not sufficiently explain the application from a consumer perspective.

Users requested simpler guidance explaining:

- What each Data Compass object means.
- How the objects relate to each other.
- How to search for data.
- How to identify the authoritative source.
- How to determine whether data is approved.
- How to request access.
- When Data Contracts should be used.
- When to use Data Compass versus Fusion.

This guidance should ideally be embedded directly into the application rather than relying only on external technical documentation.

---

# Overall Takeaway

The session showed that **the underlying metadata and governance model can identify authoritative data, but users struggle to discover it unless they already understand how the catalog is structured**.

The largest opportunity is to make CT Data Compass more:

- **Business-searchable**
- **Intuitive**
- **Self-explanatory**
- **Authoritative**
- **Actionable**

The desired user experience should be:

**“I need FARM data”**
→
**Search FARM**
→
**See the authoritative Data Product**
→
**Understand what it contains**
→
**See the approved physical tables**
→
**Understand ownership, quality, and SLA**
→
**Request access**
→
**Consume the data**

The user should not need prior knowledge of the technical implementation, organizational hierarchy, table names, CLIDs, or catalog structure to accomplish this.