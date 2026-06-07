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