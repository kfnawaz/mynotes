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