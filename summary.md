The team confirmed that Category 3 systems cannot directly read any data—including metadata—from a Category 1 PCI Databricks workspace. PCI rules differ from HCD rules, and no exception currently applies.

Data Compass only needs metadata such as table names, columns, schemas, views, and lineage, but this still counts as data movement and must follow an approved pattern from the PCI white paper. A likely solution is a controlled API or intermediary hosted in a Category 2 environment that proves no cardholder data is transmitted.

Next steps:

Review the PCI white paper and approved Category 1-to-Category 3 patterns.
Engage the Databricks team, starting with Adam Baldari, to determine what technical controls or platform changes are possible.
Schedule a joint meeting with Databricks and the PCI team.
Present the metadata-crawling requirement and proposed compliant architecture for approval.