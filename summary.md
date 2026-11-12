## Executive Summary – DataCompass Crawler Strategy

The email discussion is about reducing DataCompass's long-term dependency on Atlan crawlers and evaluating alternative solutions before Atlan's contract renewal in February 2027.

The participants generally support exploring in-house crawler development and open-source alternatives, while continuing Atlan in the short term.

A major technical concern is that extracting metadata is relatively straightforward, but extracting accurate data lineage is significantly more challenging, particularly for Oracle and complex legacy platforms.

## 1. Who is saying what?

| Person                  | Position / Feedback                                                                                                                                                                                                                                                    |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tanmay Patwardhan       | Initiated the discussion. Wants a backup strategy to reduce dependency on Atlan and improve crawler customization. Proposed four options and requested prioritization.                                                                                                 |
| Prasanth Gopalakrishnan | Recommends Option 1 (build in-house) and Option 3 (evaluate DataHub open-source crawlers). Suggests rapidly developing a proof of concept for Databricks, Snowflake, and Sigma using AI-assisted development and Rob's team.                                           |
| Haiying Guo             | Offers support from Robert Gubelic's team, specifically a DBA expert who can help research crawlers for OLTP databases. Also warns that TBPB lineage cannot be reliably extracted through crawling, based on previous attempts using IBM Manta and other vendor tools. |
| Amit Justa              | Recommends prioritizing Oracle because a large portion of the CT technology estate uses it. Emphasizes that metadata discovery is easier than lineage extraction, which requires deeper investigation.                                                                 |
| Kailashchandra Mishra   | Copied on the conversation but no direct response is included.                                                                                                                                                                                                         |

## 2. Options being considered

| Option                 | Description                                                              | Discussion outcome                                           |
| ---------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. Build internally    | Develop custom crawlers using Rob Gubelic's team                         | Recommended by Prasanth for an initial POC                   |
| 2. Partner with AMDP   | Leverage Steve M / Saurabh Gupta's team and JADE harvesting capabilities | Proposed by Tanmay; no specific response                     |
| 3. Open-source DataHub | Evaluate DataHub's existing crawlers                                     | Recommended by Prasanth; packages appear installable in JPMC |
| 4. Continue Atlan      | Retain existing Atlan crawlers                                           | Necessary in the short term; long-term decision pending      |

The numbering above follows Tanmay's original list, interpreting his three alternatives plus continuing Atlan as four options.

## 3. Key technical observations

Modern platforms (Snowflake, Databricks, Sigma): Prasanth believes custom crawlers can be developed relatively quickly and customized to meet DataCompass requirements.

Oracle / OLTP platforms: Amit believes Oracle should receive significant attention because of its prevalence in CT. Haiying has offered DBA expertise to support research.

Lineage extraction: This is the biggest technical challenge. Haiying specifically cautions that TBPB lineage resisted extraction even with established commercial products, including IBM Manta.

Customization: Regardless of which crawler technology is selected, DataCompass needs more flexibility to meet CT tower and CDO requirements for lineage automation.

## 4. What is the conclusion?

There is directional agreement, but no formally documented final decision.

The discussion points toward the following approach:

1. Short term: Continue using Atlan crawlers through the upcoming contract renewal while alternative solutions are evaluated.
2. Initial evaluation: Explore internally built crawlers and DataHub's open-source crawlers in parallel.
3. Technical POCs: Start with modern platforms such as Snowflake, Databricks, and Sigma, where development may be simpler.
4. Oracle investigation: Engage Rob Gubelic's DBA expert to evaluate Oracle metadata and lineage extraction.
5. Long-term roadmap: Decide whether to build, adopt open-source technology, partner with AMDP, or retain Atlan based on technical feasibility and cost.

No one explicitly confirmed ownership, timelines, POC acceptance criteria, or a final technology selection.

## 5. What this means for DataCompass

The important distinction is that the Atlan UI has already been replaced, but DataCompass still depends on Atlan for metadata crawling and lineage harvesting.

This conversation represents an effort to examine whether that remaining dependency can also be reduced or eliminated.

The strategic direction is:

Move toward greater ownership and customization of metadata harvesting while maintaining Atlan as an interim solution and determining which lineage requirements can realistically be supported.

One important caution: successfully developing a metadata crawler does not necessarily mean DataCompass can reproduce Atlan's lineage extraction capabilities. The POCs should evaluate both separately.

Bottom line: Tanmay wants an Atlan exit or fallback strategy; Prasanth favors internal development plus DataHub evaluation; Haiying offers technical expertise while highlighting lineage limitations; and Amit wants Oracle and lineage extraction given sufficient priority. No final decision has been recorded.