## **Objective**

Understand the background behind the Databricks centralized crawling requirement and determine whether it should be addressed as part of the current Java-based crawler development.

## **Key Findings**

- **Shared Unity Catalog metastore:** Multiple Databricks workspaces are connected to a common metastore, allowing catalogs to be bound across workspaces and environments.
- **Catalog binding:** Databricks enables catalogs to be made available across workspaces without physically copying the underlying data, supporting use cases such as reconciliation and pre-production testing.
- **Duplicate metadata:** Atlan's workspace-level crawling discovers both native and externally bound catalogs, potentially resulting in duplicate asset representations within DataCompass.
- **Catalog identification:** Changes to catalog naming conventions removed the ability to reliably derive SEAL mappings from catalog names. Workspace IDs, metastore IDs, and catalog-to-workspace mappings were explored to address this.
- **Environment mapping:** Multiple data lake environments map into three DataCompass environments, further complicating asset identification and deduplication.
- **Existing reconciliation:** The Atlan Syncer currently addresses these complexities downstream of the crawler.

## **Centralized Crawling – Challenges**

The proposed centralized crawling approach would discover metadata directly at the Unity Catalog metastore level rather than crawling individual workspaces.

However:

- The required APIs, access permissions and implementation approach have not been established.
- Obtaining even workspace-level access has historically been challenging.
- Metastore-level access would require further engagement with the Databricks platform team.
- Technical feasibility has not yet been validated.

## **Agreed Direction**

| Capability                             | Current Java Implementation       |
| -------------------------------------- | --------------------------------- |
| Workspace-level crawling               | In scope                          |
| Existing Atlan crawler parity          | In scope                          |
| Existing Atlan Syncer capabilities     | In scope                          |
| Catalog and environment mapping        | In scope, based on existing logic |
| Asset identification and deduplication | In scope, based on existing logic |
| Centralized metastore-level crawling   | Future enhancement                |

## **Next Steps**

- Continue Java-based crawler development without introducing centralized crawling as a dependency.
- Review and incorporate existing Atlan Syncer capabilities directly into the new crawler framework, where appropriate.
- Ensure functional parity with the existing end-to-end metadata ingestion process, not just the Atlan crawler.
- Treat centralized metastore-level crawling as a separate architectural initiative requiring feasibility assessment, platform engagement and access provisioning.

## **Conclusion**

**Centralized crawling is a future architectural enhancement, not a prerequisite for the current Java conversion.**

The immediate priority is to consolidate the existing Atlan crawler and Atlan Syncer capabilities into the new framework, preserving functional parity while eliminating unnecessary downstream processing wherever possible.

Centralized crawling should neither block nor expand the scope of the current implementation until its feasibility and prerequisites are established.