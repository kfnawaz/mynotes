# Data Compass / Control Plane – Feedback Summary & Next Actions

## Executive Summary

The feedback was positive about the **Control Plane concept**, but it materially changes how the initiative should be positioned.

The main message is:

> **The operational Control Plane is important, but it is not by itself the business case.**

The stronger justification is **better metadata and lineage depth, accuracy, and consistency**, with the Control Plane providing the reliable operational mechanism behind it.

The proposal should therefore become a **two-part story, roughly 50/50**:

### 1. Business / Functional Value – Better Metadata & Lineage

Achieve greater depth, accuracy, and consistency of metadata and lineage than is available today, potentially including:

- Process-level lineage
- Column/attribute-level lineage
- Sub-asset lineage
- Better representation of Databricks/Spark processes
- More complete upstream/downstream lineage
- Platform-specific metadata nuances
- Standardized representation across platforms

Establish an **opinionated, standardized crawler interface for Data Compass**, with platform-specific implementations for:

- Databricks
- Snowflake
- Other platforms in the future

### 2. Operational Value – Control Plane

Once richer and more accurate metadata can be harvested, ensure it is collected:

- Reliably
- Repeatedly
- On schedule
- With minimal manual intervention
- With appropriate monitoring and observability

The Control Plane provides:

- Workflow execution
- Scheduling
- Run-state management
- Monitoring
- Retry capability
- Logging
- Metrics
- Versioning
- Auditability
- Operational visibility

---

# What Was Validated

## Control Plane Concept

The Control Plane concept was strongly validated.

The description of the Control Plane as an:

> **Operational Command Center for trusted metadata**

was considered accurate and aligned with the desired technical direction.

The Control Plane would convert disconnected scripts and jobs into a visible and repeatable operating model.

---

## Direct Metadata Harvesting Is Technically Feasible

The prototype demonstrated that direct metadata harvesting from enterprise platforms is technically feasible.

Much of the required metadata already exists in:

- System tables
- Platform repositories
- APIs
- Metadata stores
- Platform-specific system information

The primary engineering challenge is not necessarily inventing new algorithms.

Instead, the work involves:

1. Extracting the appropriate information.
2. Understanding platform-specific metadata and lineage semantics.
3. Determining how lineage is represented by each platform.
4. Normalizing the output into a common Data Compass model.

---

# Emerging Architecture

The architecture can be represented as:

**Enterprise Platforms**

↓  

**Platform-Specific Harvesters / Crawlers**

Examples:

- Databricks Harvester
- Snowflake Harvester
- Future platform harvesters

↓

**Standardized Metadata & Lineage Model**

↓

**Control Plane**

- Workflow management
- Scheduling
- Execution
- Monitoring
- Retry
- Logging
- Metrics
- Auditability

↓

**Data Compass**

↓

**Context / Consumer Products**

The important change is that the **standardized crawler and lineage capability should become a first-class part of the architecture and pitch**, rather than presenting the Control Plane as the primary product.

---

# Key Gaps Identified

## 1. No Atlan vs. Custom Benchmark Yet

The largest gap is the absence of an objective comparison between:

- Atlan crawling
- Custom/Data Compass crawling

The prototype proves:

> **We can build a crawler.**

But it does not yet prove:

> **Our crawler provides better, deeper, or more accurate metadata and lineage than Atlan.**

That comparison is essential to justify building the capability internally.

---

## 2. Detailed Crawling Requirements Are Not Yet Defined

The team needs to clearly document what Data Compass requires that Atlan does not currently provide.

Areas mentioned include:

- Databricks process representation
- Spark-specific behavior
- Process-level lineage
- Column/attribute-level lineage
- Sub-asset lineage
- Transformation relationships
- Multi-hop lineage
- Lineage depth
- Lineage accuracy
- Metadata freshness
- Platform-specific nuances

These requirements should become the basis for evaluating all crawler options.

---

## 3. Fusion / Jade Crawler Overlap Must Be Understood

The Fusion team appears to be considering the existing **Jade crawler** as a foundation for an improved enterprise crawler capability.

Steve McPherson's organization is expected to take the existing Jade crawler and make it "better."

However, what "better" means has not yet been clearly defined.

Before building an independent crawler, the team needs to understand:

- What Jade currently crawls
- Which platforms Jade supports
- How Jade extracts metadata
- How Jade represents lineage
- What its target architecture is
- What improvements Fusion is planning
- How Fusion intends to consume the crawler output
- Whether Data Compass requirements overlap with that work

---

# Next Action Items

| Priority | Action | Expected Output |
|---|---|---|
| **1** | **Benchmark custom crawling against Atlan** | Select representative Databricks assets and crawl exactly the same objects using both approaches. Compare depth, accuracy, processes, columns, and upstream/downstream lineage. |
| **2** | **Find known cases where custom crawling outperformed Atlan** | Reconstruct the earlier example where Atlan failed to discover lineage but the custom crawler found it. Capture screenshots and supporting evidence. |
| **3** | **Define detailed crawling requirements** | Document exactly what Data Compass requires from a crawler, including Databricks/Spark nuances, processes, columns, sub-assets, transformations, lineage depth, accuracy, and freshness. |
| **4** | **Create the standardized crawler contract** | Define the common metadata/lineage schema that Databricks, Snowflake, and future platform adapters must produce. |
| **5** | **Investigate the Fusion/Jade crawler initiative** | Meet with the Fusion/Jade team and understand architecture, scope, roadmap, interfaces, supported platforms, and planned improvements. |
| **6** | **Perform Jade vs. Atlan vs. Custom gap analysis** | Map Data Compass requirements against all three approaches and identify capabilities, overlaps, and gaps. |
| **7** | **Make a Build / Reuse / Collaborate decision** | Determine whether Data Compass should use Jade/Fusion, extend it, jointly develop it, build independently, or adopt a hybrid model. |
| **8** | **Rework the Control Plane pitch** | Change the story to approximately 50% metadata/lineage depth and accuracy and 50% operational Control Plane/reliability. |
| **9** | **Update the demo/video** | Start with the business problem and lineage improvement, then show the Control Plane as the mechanism for operationalizing the solution. |
| **10** | **Prepare for the Atlan renewal decision** | Determine which Atlan capabilities remain necessary, what can be replaced, whether parallel operation is required, and the potential transition timeline. |

---

# Most Important Immediate Deliverable

The next concrete piece of work should be the:

## Atlan vs. Custom Crawler Benchmark

Select approximately **5–10 representative Databricks examples** containing increasingly difficult lineage scenarios.

Run the same assets through both approaches and compare the results.

Example comparison:

| Capability | Atlan | Custom Crawler | Gap / Improvement |
|---|---|---|---|
| Table discovery | ✓ | ✓ | Equivalent |
| Column metadata | ✓ | ✓ | TBD |
| Table-level lineage | ✓ | ✓ | Compare accuracy |
| Column-level lineage | TBD | TBD | Benchmark |
| Process representation | TBD | TBD | Key requirement |
| Spark-specific lineage | TBD | TBD | Investigate |
| Multi-hop lineage | TBD | TBD | Compare depth |
| Missing/broken lineage | Capture examples | Capture examples | Identify custom improvements |
| Metadata freshness | Measure | Measure | Compare |
| Standardized Compass schema | Transformation required | Target model | Assess |

This changes the conversation from:

> **"We proved that we can build a crawler."**

to:

> **"Here is measurable evidence showing where controlling our own crawler provides additional metadata depth, accuracy, or capabilities."**

---

# Three-Way Technology Assessment

The work is effectively heading toward a comparison of three approaches.

## 1. Atlan

Determine:

- What Atlan provides today
- Where Atlan performs well
- Where lineage or metadata gaps exist
- What capabilities still require Atlan
- What capabilities justify continuing the license
- How difficult those capabilities would be to replace

---

## 2. Jade / Fusion

Determine:

- What crawler capability already exists
- Which platforms it supports
- What Fusion plans to enhance
- What its standardized output looks like
- Whether Data Compass can consume it
- Whether Data Compass requirements can influence its roadmap
- Whether collaboration avoids duplicate engineering

---

## 3. Data Compass / Custom

Determine:

- What requirements cannot be satisfied by Atlan
- What requirements cannot be satisfied by Jade/Fusion
- What should therefore be built specifically for Data Compass
- Whether custom platform adapters are necessary
- What should be owned directly by the Data Compass team

---

# Key Architecture Decision

Once the requirements and benchmarks are available, the team should answer:

> **Build, reuse, extend, collaborate, or use a hybrid approach?**

Possible target direction:

**Atlan**

↓  

**Hybrid / Parallel Operation**

↓  

**Jade/Fusion + Data Compass Custom Capabilities**

↓  

**Progressively replace selected Atlan capabilities where justified**

This does **not** necessarily mean Atlan should immediately be replaced.

The evidence should determine:

- What remains in Atlan
- What moves in-house
- What can leverage Jade/Fusion
- What Data Compass needs to build
- How long parallel operation is required

---

# Atlan Renewal as the Decision Milestone

The Atlan renewal timeframe creates an important decision point.

The work completed before the renewal decision should provide enough evidence to answer:

1. What are we renewing Atlan for?
2. Which capabilities still depend on Atlan?
3. Which capabilities can already be provided internally?
4. What can Jade/Fusion provide?
5. What still needs to be built?
6. How long do we need Atlan?
7. Should Atlan and the internal solution run in parallel?
8. What conditions would allow capabilities to transition away from Atlan?

Therefore, the period leading up to the renewal should be treated as an:

> **Evidence-Gathering and Architecture-Decision Phase**

rather than immediately committing to a full custom implementation.

---

# Recommended Revised Pitch

The pitch should change from:

> **"We built a Control Plane to reliably operate metadata pipelines."**

to:

> **"Data Compass needs richer, more accurate, and more standardized metadata and lineage than we can consistently obtain today. By controlling the crawling and normalization layer, we can capture the required level of detail across platforms. The Control Plane then ensures that this trusted metadata is produced reliably, repeatedly, observably, and at enterprise scale."**

The story becomes:

### Part 1 — Why Build It?

**Metadata and lineage depth, accuracy, and standardization**

### Part 2 — How Do We Operate It?

**Control Plane automation, reliability, monitoring, and governance**

---

# Bottom Line

The strongest takeaway from the discussion is:

> **Don't make the Control Plane the primary reason to fund the initiative. Make richer, more accurate, standardized metadata and lineage the business reason—and make the Control Plane the mechanism that ensures that capability runs reliably at enterprise scale.**

The immediate focus should therefore be:

**Requirements → Benchmark → Jade/Fusion Assessment → Gap Analysis → Architecture Decision → Atlan Renewal Decision**

rather than moving directly from the prototype into a full custom build.