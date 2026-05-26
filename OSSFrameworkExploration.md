The OSS Framework Exploration
Let me cover OpenMetadata and DataHub: what each is, how their ingestion frameworks work, how they'd interact with the contracts just defined, and the specific trade-offs for your situation.

OpenMetadata vs. DataHub — Framework Evaluation
What You're Actually Evaluating
Both are complete open-source data catalog platforms — they have their own metadata models, lineage engines, UIs, and APIs. You are not adopting their full stack. The evaluation question is narrower: can you use their ingestion/connector layer as the implementation of the MetadataConnector interface above, while discarding everything else?
OpenMetadata
What it is: A fully open-source catalog (Apache 2.0) with a clean ingestion framework. The ingestion layer is a standalone Python package (openmetadata-ingestion) that runs independently of the OpenMetadata server.
How ingestion works:
Source → Stage → Sink
Each connector is a Source class that yields Entity objects. A Sink writes them somewhere — by default to the OpenMetadata server, but you write a custom Sink that maps to your CatalogDistribution/CatalogAttribute models and writes to MongoDB instead.
Databricks connector coverage:

Unity Catalog tables, views, columns — yes, via DatabricksSource
Descriptions and column comments — yes
Lineage from query history — yes, via a separate DatabricksLineageSource
table_type, created, last_altered — check against live connector code (changes frequently)

Fit against your interface contracts:

The OpenMetadata Source essentially implements the spirit of MetadataConnector — you'd wrap it to satisfy the Protocol formally
You'd write a custom Sink implementing CatalogLoader
The normalization layer is mostly yours — OpenMetadata normalizes into its model, not yours; you map from OpenMetadata's entity shape into Raw* models or directly into Catalog*
JRN minting, environment lookup, dataType enum normalization — all yours regardless

Strengths for your project:

Connector handles UC pagination, rate limiting, auth (OAuth / PAT / service principal) — already solved
Active maintenance — Databricks pushes changes to UC APIs; the OSS community absorbs them
openmetadata-ingestion is pip-installable; no need to run OpenMetadata's server
Well-documented connector configuration

Weaknesses:

OpenMetadata's internal metadata model is not your model — you will write a non-trivial mapping layer
The connector emits OpenMetadata entity types (Table, Column, DatabaseSchema) — you translate these to RawDistribution, RawAttribute yourself, or skip the raw layer and map straight to Catalog*
Connector output shape changes between OpenMetadata releases — you carry a framework upgrade dependency
Some connector behaviors are opinionated (e.g. how qualified names are constructed) — may conflict with how your JRN minting service expects inputs

DataHub
What it is: LinkedIn-originated, open-source (Apache 2.0). More enterprise-oriented; larger install base in financial services. The ingestion layer is also a standalone Python package (acryl-datahub).
How ingestion works:
Source → MetadataChangeProposalWrapper (MCP) → Sink
Connectors emit MetadataChangeProposal events — DataHub's own change-event format. A RestSink pushes to DataHub's GMS server; you write a custom sink that intercepts MCPs and maps to your models.
Databricks connector coverage:

Unity Catalog tables, views, columns — yes
Lineage — yes, from system.access.* (DataHub's Databricks connector uses the same UC system tables identified in the extraction query plan)
Column-level lineage — yes
Ownership, tags — yes

Fit against your interface contracts:

DataHub's MCP format is richer but more complex than OpenMetadata's entities — more translation work
The custom sink pattern works the same way — you intercept MCPs before they reach DataHub's server
Like OpenMetadata, JRN minting, environment lookup, and dataType normalization are entirely yours

Strengths for your project:

Extremely mature Databricks connector — DataHub is widely used in financial services specifically
Column-level lineage from Unity Catalog system.access.column_lineage is already implemented — the hardest part of Workstream 2 is largely solved
acryl-datahub is pip-installable; server not required
The MCP event model maps naturally to a delta/change-event ingestion pattern

Weaknesses:

The MCP abstraction layer adds complexity between the connector and your models compared to OpenMetadata's simpler entity output
DataHub's qualified name construction (urn:li:dataset:(urn:li:dataPlatform:databricks,...)) is opinionated — JRN minting service needs to accept or transform these inputs
Larger dependency footprint than OpenMetadata

Head-to-Head for Your Situation
DimensionOpenMetadataDataHubGreenfieldDatabricks crawler maturityGoodExcellentYou build itColumn-level lineage (UC)GoodExcellent — already uses system.access.*You build itCustom sink / custom catalog supportClean — simple entity modelWorks — more translationNativeMapping effort to your modelModerateModerate-high (MCP format)None — you own the modelConnector maintenance absorbedYesYesYou own foreverFramework upgrade dependencyYesYesNoneFinancial services adoptionGrowingEstablishedN/Apip-installable, no server neededYesYesN/A
Recommendation for Phase 0 Spike
Run both against a real Databricks workspace. Specifically measure:

What DatabricksSource / DataHub's Databricks connector actually emits for a Unity Catalog table — check every field your parity matrix requires is present and correctly valued.
How much mapping code the spike requires to go from the framework's output to a CatalogDistribution document in MongoDB. That mapping code is the real cost of adoption.
Lineage quality — does DataHub's connector produce column-level lineage edges from system.access.column_lineage that map cleanly to your DataFlow model?
Qualified name construction — how does each framework construct its canonical identifier, and does it give the JRN minting service what it needs as input?

If the mapping code in the spike is small (a thin translation layer), adopt the framework — you get connector maintenance and lineage for free.
If the mapping code is large (re-implementing most of what the connector produces anyway), the framework is adding complexity without proportional value — build the MetadataConnector implementation directly against the UC SQL APIs defined in the extraction query plan.
The interface contracts already written mean the spike's output is a concrete build estimate: how many lines of code does a MetadataConnector implementation take against each framework vs. greenfield? That number makes the decision objective.