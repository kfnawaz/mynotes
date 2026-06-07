Altery workflows are not tables/views. How should RawDistribution.object_type be set?
The SourceTypeName enum currently has TABLE | VIEW | COLUMN | SCHEMA | CATALOG. Tableau uses SourceTypeName. TABLE for datasources as a pragmatic workaround. S10b proposes "WORKFLOW" but that value doesn't exist.
1 Use SourceTypeName.TABLE (same as Tableau)
No model change, zero downstream impact - workflows masquerade as TABLE in the type system
2 Add WORKFLOW to SourceTypeName enum
Semantically correct, small model change, needs downstream handling in normalizer.py (~1 line)
3 Comments here


Option 2 — add WORKFLOW to the enum.
Here's why:
The catalog's job is to tell you what something is. A workflow appearing as TABLE forces every downstream consumer to look at the platform field to figure out it's actually a workflow. That defeats the purpose of having a type system.
The cost is trivial. One line in the enum, one line in each normalizer's type mapping. No downstream handling is actually needed — the normalizer already produces the string, the load layer writes whatever string it gets, MongoDB stores it, and the UI renders it. Nothing in the pipeline switches on SourceTypeName values.
The Tableau workaround was wrong, not a precedent. Datasources aren't tables either. The right fix is to add DATASOURCE for Tableau at the same time — two enum values, two normalizer lines, zero downstream risk. But that's a separate cleanup; don't let it block S10b.
Future-proofing. You'll eventually catalog APIs, dashboards, reports, notebooks. Each needs a type value. Starting to use TABLE as a catch-all makes the enum meaningless.
The change:
python# models/shared.py
class SourceTypeName(str, Enum):
    TABLE = "TABLE"
    VIEW = "VIEW"
    MATERIALIZED_VIEW = "MATERIALIZED_VIEW"
    COLUMN = "COLUMN"
    SCHEMA = "SCHEMA"
    CATALOG = "CATALOG"
    WORKFLOW = "WORKFLOW"          # ← add for Alteryx
    # DATASOURCE = "DATASOURCE"   # ← consider for Tableau (separate PR)
One enum value now. Fix Tableau's DATASOURCE in a follow-up if you want — it's not blocking anything.



What should the JRN path look like for an Alteryx workflow?
V
JrnBuilder.mint_distribution_jrn always produces platform/catalog_name/schema_name/object_name. For Alteryx the natural mapping is catalog name=server _host, schema_name=project_name, object_name=workflow_name → alteryx/(server)/(project}/{workflow} (4 levels, same depth as Tableau site/project/datasource). The S10b doc shows alteryx/{server}/{workflow} (3 levels, skipping project) which would require schema_name="" and is inconsistent.
1 4-level: alteryx/(server)/(project)/[workflow}
Consistent with JrnBuilder contract and Tableau pattern - server=catalog, project=schema, workflow=object
2 3-level: alteryx/{server)/(workflow) (doc as-written)
Requires schema_name=" and collapses project into workflow name — loses project hierarchy
3 custom answer here .

Show less
2:30 AM

Evaluated JRN path consistency against JrnBuilder contract and existing patterns
Evaluated JRN path consistency against JrnBuilder contract and existing patterns
Option 1 — four levels: alteryx/{server}/{project}/{workflow}.

The mapping is exact:

Concept	Databricks	Snowflake	Tableau	Alteryx
catalog_name	catalog	database	site	server
schema_name	schema	schema	project	project
object_name	table	table	datasource	workflow
JRN path	databricks/cat/sch/tbl	snowflake/db/sch/tbl	tableau/site/proj/ds	alteryx/srv/proj/wf
All four levels, same depth, same JrnBuilder.mint_distribution call, zero special handling needed.

The 3-level version from the S10b doc was wrong — it loses the project hierarchy, which matters for scope filtering (include/exclude_projects) and organizational context. I'll update the doc.


How should Altery project-level scope filtering be expressed in config?
The S10b doc introduces new include_projects/exclude_projects fields. But CrawlScope already handles Tableau project filtering through
include_schemas/exclude_schemas (project = schema convention). Adding new fields means extending the shared CrawlScope model.

﻿﻿Reuse include_schemas / exclude_schemas for projects Zero CrawlScope changes — same convention as Tableau. Config users write 'schema' but mean 'project".
﻿﻿Add include_ projects / exclude_projects to CrawlScope Explicit and self-documenting, but adds 2 fields to shared model used by all platforms
2:37 AM

Discovered that project fields already exist in CrawlScope
Discovered that project fields already exist in CrawlScope
Neither — include_projects / exclude_projects already exist in CrawlScope.

They were added during the scope filtering story for Tableau. Check:

bash
grep -n "include_projects\|exclude_projects" src/datacompass_metadata_harvester/models/shared.py
Alteryx just uses them — same fields, same glob matching, zero model changes. The Tableau connector already filters on these fields in get_distribution_inventory(). The Alteryx connector does the same:

python
# connectors/alteryx/connector.py — already works
if not ScopeFilter.is_included(
    project_name,
    self._scope.include_projects,
    self._scope.exclude_projects,
):
    continue
Config is self-documenting because the field is already called projects, not schemas:

yaml
alteryx_workspaces:
  - workspace_name: alteryx-prod
    scope:
      exclude_projects:        # ← same field Tableau uses
        - "*_test"
        - "*_sandbox"
        - "*_dev"
No new fields, no model change, no Option 1 workaround. The S10b doc was wrong to suggest adding new fields — they already exist.




How should the Alteryx miner resolve workflow 1/0 connections to JRNs?
The S10b miner plans to use ConnectionRegistry. resolve (namespace, name) - designed for OpenLineage namespace strings like snowflake://account.snowflakecomputing.com. com. But Alteryx One's lineage API gives structured data: {"connectionType": "ODBC", "server":
"...", "database": "...", "schema": "...", "table": "..."}. Two options:﻿﻿﻿﻿Synthetic namespace (jdbc: odbc: //server/database): construct a namespace string from connection data, add Alteryx-specific patterns to connection_registry in config. Keeps ConnectionRegistry as single resolver but needs config entries per Altery ODBC source.﻿﻿﻿﻿Direct JRN construction: bypass ConnectionRegistry, look up catalog_seal_map by server/database name and call IrnBuilder mint_distribution_jrn() directly. Simpler code, no connection registry config needed, but diverges from the S10a OpenLineage receiver approach. 1 B - Direct JRN from structured connection data Simpler miner code: Alteryx structured data maps cleanly to mint_distribution imO params
2 A - Synthetic namespace via ConnectionRegistry
Consistent with $10a receiver; config entries enable/disable per-source JRN resolution
3 custom answer hereShow less2:49 AMClaude responded: Option 1 — synthetic namespace via ConnectionRegistry.Weighed architectural tradeoffs between registry consistency and direct constructionWeighed architectural tradeoffs between registry consistency and direct constructionOption 1 — synthetic namespace via ConnectionRegistry.
The reason is seal_node_id. To construct a JRN for snowflake/PROD_DB/risk/positions, you need seal_node_id 34243 — which lives in the Snowflake workspace's catalog_seal_map, not in the Alteryx config. Direct JRN construction (Option 2) would require duplicating every source platform's seal_node_id mapping into the Alteryx workspace config. That's the wrong place for it — the ConnectionRegistry already has it.
The synthetic namespace construction is a 5-line helper, not real complexity:
python_CONN_TYPE_TO_SCHEME: dict[str, str] = {
    "odbc": "mssql",
    "snowflake": "snowflake",
    "oracle": "oracle",
    "teradata": "teradatasql",
    "databricks": "databricks",
}

def _to_namespace(connection: dict) -> str:
    """Convert Alteryx structured connection to OpenLineage namespace."""
    scheme = _CONN_TYPE_TO_SCHEME.get(
        connection.get("connectionType", "").lower(), "unknown"
    )
    server = connection.get("server", "")
    database = connection.get("database", "")
    return f"{scheme}://{server}/{database}"
Then the miner calls:
pythonnamespace = _to_namespace(input_connection)
name = f"{input_connection['schema']}.{input_connection['table']}"
resolved = self._connection_registry.resolve(namespace, name)
Same registry, same config entries, same resolution path as S10a. The S10a receiver's connection_registry config entries already cover all the source systems — the miner reuses them with zero additional config.
The alternative (Option 2) looks simpler in the miner code but pushes complexity into config management — every Alteryx workspace config would need a copy of every source platform's seal_node_id mapping, and those copies would need to be kept in sync. That's an operational burden that ConnectionRegistry was specifically designed to eliminate.