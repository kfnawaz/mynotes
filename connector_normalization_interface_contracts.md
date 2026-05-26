# Connector / Normalization Interface Contracts

**Project:** In-House Replacement of Atlan Crawlers/Miners
**Purpose:** Defines the code-level interfaces every platform connector and the normalization layer must implement. These contracts are the stable boundary between platform-specific extraction and the Catalog DB. Adding a new platform means implementing these interfaces — nothing downstream changes.
**Companion artifacts:** `atlan_replacement_proposal.md`, `databricks_uc_extraction_query_plan.md`, `mvp_parity_matrix.md`, `data_compass_model.md`.

> **Language note:** Interfaces below are expressed in Python (type-annotated, Protocol-based). If the implementation language differs, translate the contracts — the interface shapes and invariants are the deliverable, not the syntax.

---

## 1. Design Principles

1. **Connectors are dumb.** They extract and emit raw platform objects. They do not transform, normalize, or write to the catalog. If a connector contains catalog model knowledge, it is wrong.
2. **Normalization is stateless per record.** Given a raw platform object, the normalizer always produces the same catalog document. Side effects (JRN minting service calls, lookup table reads) are injected dependencies, not hidden state.
3. **All contracts are pull-based.** Connectors yield records; the pipeline pulls them. No connector pushes to any downstream system directly.
4. **Contracts are versioned.** Any breaking change to an interface requires a version bump and a migration plan. Additive changes (new optional fields on a raw model) are non-breaking.
5. **Failure is explicit.** Connectors surface errors per-record via a typed result, never silently drop records. The pipeline decides what to do with errors.

---

## 2. Shared Types

Used across all interfaces.

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from typing import Generic, Iterator, Optional, TypeVar


# ---------------------------------------------------------------------------
# Enumerations
# ---------------------------------------------------------------------------

class Platform(str, Enum):
    DATABRICKS = "DATABRICKS"
    SNOWFLAKE  = "SNOWFLAKE"
    TABLEAU    = "TABLEAU"
    # extend as platforms are added


class SyncStatus(str, Enum):
    IN_PROGRESS = "IN_PROGRESS"
    PROCESSED   = "PROCESSED"
    FAILED      = "FAILED"


class AssetStatus(str, Enum):
    ACTIVE  = "ACTIVE"
    DELETED = "DELETED"


class SourceTypeName(str, Enum):
    TABLE   = "TABLE"
    VIEW    = "VIEW"
    COLUMN  = "COLUMN"
    SCHEMA  = "SCHEMA"
    CATALOG = "CATALOG"


# Model data-type enum (from data_compass_model.md)
class DataTypeEnum(str, Enum):
    BOOLEAN   = "BOOLEAN"
    CHAR      = "CHAR"
    STRING    = "STRING"
    DATE      = "DATE"
    TIME      = "TIME"
    TIMESTAMP = "TIMESTAMP"
    INTEGER   = "INTEGER"
    LONG      = "LONG"
    DOUBLE    = "DOUBLE"
    FLOAT     = "FLOAT"
    DECIMAL   = "DECIMAL"
    UUID      = "UUID"
    JSON      = "JSON"
    XML       = "XML"
    BYTE      = "BYTE"
    BINARY    = "BINARY"
    ARRAY     = "ARRAY"
    OTHER     = "OTHER"


# ---------------------------------------------------------------------------
# Provenance block — written to every catalog document (source.*)
# ---------------------------------------------------------------------------

@dataclass
class SourceProvenance:
    """
    The platform-agnostic provenance sub-document written to every
    DataDistribution and DataAttribute MongoDB document as `source.*`.

    During the dual-write transition window the load layer also writes the
    legacy `atlan.*` field names with the same values. Once all readers
    migrate to `source.*` the legacy block is dropped.
    """
    source_system:          Platform              # constant per connector
    source_guid:            str                   # platform-native object identity
    source_qualified_name:  str                   # fully-qualified platform name
    source_type_name:       SourceTypeName
    source_status:          AssetStatus
    source_create_time:     Optional[datetime]
    source_update_time:     Optional[datetime]
    sync_status:            SyncStatus
    sync_time:              datetime              # set by load layer at write time


# ---------------------------------------------------------------------------
# Typed result wrapper — connectors never raise on per-record errors
# ---------------------------------------------------------------------------

T = TypeVar("T")

@dataclass
class ExtractionResult(Generic[T]):
    """
    Wraps a single extracted record. Either `record` is set (success)
    or `error` is set (failure). Never both. Never neither.
    """
    record:     Optional[T]   = None
    error:      Optional[str] = None
    source_ref: str           = ""   # the FQN or identifier that failed, for logging

    @property
    def is_ok(self) -> bool:
        return self.record is not None and self.error is None
```

---

## 3. Connector Contract

Every platform connector implements the `MetadataConnector` Protocol. The pipeline interacts with connectors only through this interface.

### 3.1 Connection Config

```python
@dataclass
class ConnectorConfig:
    """
    Platform-agnostic connection configuration. One ConnectorConfig
    per crawlable INSTANCE — a Databricks workspace, a Snowflake
    instance, or a Tableau Online account. Populated per instance from
    the instance registry. The same connector code serves every
    instance of a platform; only this config differs.

    Platform-specific values go in `extra` — the connector reads what it needs
    (e.g. Databricks SQL warehouse id, Snowflake role + warehouse, Tableau site).
    """
    platform:         Platform
    workspace_name:   str           # instance identifier; becomes connectionName in catalog
    environment:      str           # PROD / UAT / DEV — resolved per instance before connector init
    secret_name:      str           # Kubernetes Secret name holding this instance's credentials
    extra:            dict          # platform-specific: host, warehouse_id, account, role, site, etc.
    scope:            "CrawlScope"


@dataclass
class CrawlScope:
    """
    Defines which objects the connector should include/exclude.
    The connector must respect these filters — it must not emit
    objects outside scope even if the platform returns them.
    """
    include_catalogs:  Optional[list[str]] = None  # None = all
    exclude_catalogs:  list[str]           = field(default_factory=list)
    include_schemas:   Optional[list[str]] = None  # None = all
    exclude_schemas:   list[str]           = field(default_factory=list)
    # Always exclude internal system schemas
    # e.g. information_schema, system catalogs
    # Connectors apply this automatically.
```

### 3.2 `MetadataConnector` Protocol

> **Swappability — acquisition vs. raw mapping.** A connector implementation has two internal sub-roles: **acquisition** (talks to the platform — greenfield SQL/API calls, or an OSS framework's ingestion `Source`) and **raw mapping** (converts acquired data into the `Raw*` models below). Together they satisfy this Protocol. The acquisition sub-role is replaceable per platform — a greenfield connector and an OSS-framework-backed connector (e.g. DataHub) are interchangeable as long as both satisfy `MetadataConnector` and emit valid `Raw*` models. Downstream layers depend only on this Protocol and the `Raw*` shapes — never on the acquisition implementation. See the architecture document §2.1.1.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class MetadataConnector(Protocol):
    """
    The interface every platform connector must implement.

    Connectors are stateless between calls except for the connection
    itself. They yield raw platform objects — no normalization, no
    catalog model knowledge, no writes.
    """

    # ------------------------------------------------------------------
    # Lifecycle
    # ------------------------------------------------------------------

    def connect(self) -> None:
        """
        Establish and validate the connection to the platform.
        Raise ConnectorConnectionError on failure.
        Must be called before any extraction method.
        """
        ...

    def disconnect(self) -> None:
        """
        Release all platform connections and resources cleanly.
        Must be idempotent — safe to call multiple times.
        """
        ...

    # ------------------------------------------------------------------
    # Inventory methods (lightweight — FQN + last-modified only)
    # Used by delta logic to detect creates, updates, and deletes
    # without pulling full attribute detail.
    # ------------------------------------------------------------------

    def get_distribution_inventory(self) -> Iterator[ExtractionResult["RawDistributionSummary"]]:
        """
        Yield one summary record per in-scope table/view.
        Must include at minimum: fully-qualified name + last-modified timestamp.
        Must complete within one crawl run — no partial inventories.
        """
        ...

    def get_attribute_inventory(self) -> Iterator[ExtractionResult["RawAttributeSummary"]]:
        """
        Yield one summary record per in-scope column.
        """
        ...

    # ------------------------------------------------------------------
    # Detail extraction methods (full attributes)
    # Called only for objects flagged by delta logic as changed/new.
    # ------------------------------------------------------------------

    def get_distribution_detail(
        self, fqn: str
    ) -> ExtractionResult["RawDistribution"]:
        """
        Return full attribute detail for a single table/view identified by fqn.
        fqn format is platform-specific (e.g. catalog.schema.table for UC).
        """
        ...

    def get_attribute_details(
        self, distribution_fqn: str
    ) -> Iterator[ExtractionResult["RawAttribute"]]:
        """
        Yield full attribute detail for all columns of a given table/view.
        Batched per parent object — the pipeline calls this once per changed table.
        """
        ...

    # ------------------------------------------------------------------
    # Lineage extraction (Workstream 2)
    # ------------------------------------------------------------------

    def get_lineage_events(
        self, since: datetime
    ) -> Iterator[ExtractionResult["RawLineageEvent"]]:
        """
        Yield raw lineage observations recorded since `since` (watermark-based).
        `since` is the timestamp of the last successfully processed lineage event.
        Must be safe to re-call with the same `since` — idempotent reads.
        """
        ...

    # ------------------------------------------------------------------
    # Metadata
    # ------------------------------------------------------------------

    @property
    def platform(self) -> Platform:
        """Return the Platform enum value this connector serves."""
        ...

    @property
    def connector_version(self) -> str:
        """Semantic version string of this connector implementation."""
        ...
```

### 3.3 Connector Exceptions

```python
class ConnectorError(Exception):
    """Base class for all connector errors."""

class ConnectorConnectionError(ConnectorError):
    """Raised when connect() fails to establish or validate the connection."""

class ConnectorPermissionError(ConnectorError):
    """Raised when the service principal lacks required permissions."""

class ConnectorQueryError(ConnectorError):
    """Raised on a platform query failure that cannot be retried."""
```

---

## 4. Raw Platform Models

The typed output of each connector method. These are platform-agnostic shapes that the normalization layer consumes. Connectors map their platform-specific API/SQL responses into these models.

> **These models are the stability contract for extraction-layer swappability.** The `MetadataConnector` methods are easy to swap; the `Raw*` model *shape* is the load-bearing contract. To keep the extraction layer replaceable (greenfield ↔ OSS-framework-backed, per platform): keep these models **tool-neutral** — they describe a table, a column, a lineage edge, never "what Databricks SQL returns" or "what DataHub emits"; add new fields as **optional**; route implementation-specific extras through the `properties: dict` catch-all rather than adding fields. Adding a *required* field here is a breaking change (§10) and must be a deliberate, versioned decision — never an incidental result of swapping an extraction implementation.

### 4.1 Distribution (Table / View)

```python
@dataclass
class RawDistributionSummary:
    """
    Lightweight — FQN + timestamp only.
    Used by delta logic in get_distribution_inventory().
    """
    platform:       Platform
    fqn:            str           # canonical fully-qualified name
    last_modified:  Optional[datetime]


@dataclass
class RawDistribution:
    """
    Full table/view detail. Output of get_distribution_detail().
    All platform connectors populate this same shape.
    """
    platform:            Platform
    fqn:                 str        # e.g. catalog.schema.table (UC)
                                    #      database.schema.table (Snowflake)

    # Identity
    platform_object_id:  Optional[str]   # platform-native ID if available (e.g. table_id)
    workspace_name:      str             # becomes connectionName

    # Core metadata
    catalog_name:        Optional[str]   # UC: catalog name; Snowflake: database name
    schema_name:         str
    object_name:         str
    object_type:         SourceTypeName  # TABLE or VIEW
    description:         Optional[str]
    owner:               Optional[str]   # platform-reported owner if available

    # Timestamps
    created_at:          Optional[datetime]
    last_modified:       Optional[datetime]

    # Extended (populated when available — not required for MVP)
    format:              Optional[str]   # e.g. DELTA, PARQUET, CSV
    location:            Optional[str]   # storage path for external tables
    view_definition:     Optional[str]   # SQL text for views
    properties:          dict = field(default_factory=dict)  # any extra k/v
```

### 4.2 Attribute (Column)

```python
@dataclass
class RawAttributeSummary:
    """Lightweight — FQN + parent FQN only. Used by delta inventory."""
    platform:         Platform
    fqn:              str          # e.g. catalog.schema.table.column
    parent_fqn:       str          # parent distribution FQN
    last_modified:    Optional[datetime]


@dataclass
class RawAttribute:
    """
    Full column detail. Output of get_attribute_details().
    """
    platform:             Platform
    fqn:                  str
    parent_fqn:           str         # parent distribution FQN

    # Identity
    platform_object_id:   Optional[str]

    # Core metadata
    column_name:          str
    raw_data_type:        str         # raw platform type string — normalizer maps to enum
    description:          Optional[str]
    ordinal_position:     Optional[int]

    # Column characteristics
    is_nullable:          Optional[bool]
    is_primary_key:       Optional[bool]
    is_foreign_key:       Optional[bool]
    is_unique:            Optional[bool]
    is_auto_incremented:  Optional[bool]
    is_generated:         Optional[bool]

    # Numeric precision
    precision:            Optional[int]
    scale:                Optional[int]
    max_length:           Optional[int]

    # Extended
    default_value:        Optional[str]
    properties:           dict = field(default_factory=dict)
```

### 4.3 Lineage Event

```python
@dataclass
class RawLineageEvent:
    """
    A single observed lineage event. Output of get_lineage_events().
    Normalizer aggregates these into DataFlow / DataProcess entities.
    """
    platform:           Platform
    event_id:           str               # platform event identifier
    event_timestamp:    datetime

    # Source and target — FQN to catalog-level (table or column)
    source_fqn:         str               # fully-qualified source object
    target_fqn:         str               # fully-qualified target object
    source_column_fqn:  Optional[str]     # if column-level lineage available
    target_column_fqn:  Optional[str]

    # Producing entity
    producing_entity_type:  Optional[str] # JOB, NOTEBOOK, QUERY, PIPELINE, etc.
    producing_entity_id:    Optional[str]
    producing_entity_name:  Optional[str]

    # Precision
    is_column_level:    bool = False

    properties:         dict = field(default_factory=dict)
```

---

## 5. Normalization Contract

The normalization layer consumes `Raw*` models and produces `Catalog*` models ready to write to MongoDB. It is platform-agnostic — it reads from the raw models, applies the Catalog DB model's rules, and calls injected services (JRN minting, environment lookup, `dataType` mapping).

### 5.1 Injected Dependencies

```python
from typing import Protocol

class JrnMintingService(Protocol):
    """
    Interface to the existing JRN minting service.
    The normalizer calls this — never constructs JRNs itself.
    """
    def mint_distribution_jrn(
        self,
        platform: Platform,
        workspace_name: str,
        catalog_name: Optional[str],
        schema_name: str,
        object_name: str,
    ) -> str:
        """
        Returns the JPMorgan Resource Name for a Distribution asset.
        Must be idempotent — same inputs always return the same JRN.
        """
        ...

    def mint_attribute_jrn(
        self,
        parent_distribution_jrn: str,
        column_name: str,
    ) -> str:
        """
        Returns the JRN for an Attribute, scoped under its parent Distribution JRN.
        Must be idempotent.
        """
        ...


class EnvironmentLookup(Protocol):
    """
    Resolves a Databricks workspace name to an environment label.
    """
    def resolve(self, workspace_name: str) -> str:
        """
        Returns e.g. 'PROD', 'UAT', 'DEV'.
        Raises EnvironmentResolutionError if workspace_name is not in the lookup.
        Unmapped workspaces must fail loudly — never silently default.
        """
        ...


class DataTypeMapper(Protocol):
    """
    Maps a raw platform data type string to the model's DataTypeEnum.
    """
    def map(self, raw_type: str, platform: Platform) -> DataTypeEnum:
        """
        Returns DataTypeEnum.OTHER for unrecognized types — never raises.
        Logs a warning when OTHER is returned so the mapping table can be updated.
        """
        ...


class EnvironmentResolutionError(Exception):
    """Raised by EnvironmentLookup.resolve() for an unmapped workspace."""
```

### 5.2 `Normalizer` Protocol

```python
@runtime_checkable
class Normalizer(Protocol):
    """
    Converts a raw platform model into a catalog document ready for the load layer.
    Stateless per call — no side effects except calls to injected services.
    """

    def normalize_distribution(
        self,
        raw: RawDistribution,
        status: AssetStatus,
    ) -> "CatalogDistribution":
        """
        Produces a CatalogDistribution document from a RawDistribution.
        `status` is supplied by the delta layer (ACTIVE for present, DELETED for absent).
        """
        ...

    def normalize_attribute(
        self,
        raw: RawAttribute,
        parent_jrn: str,
        status: AssetStatus,
    ) -> "CatalogAttribute":
        """
        Produces a CatalogAttribute document from a RawAttribute.
        `parent_jrn` is the already-minted JRN of the parent Distribution.
        """
        ...

    def normalize_lineage_event(
        self,
        raw: RawLineageEvent,
    ) -> "CatalogLineageEvent":
        """
        Produces a CatalogLineageEvent from a RawLineageEvent.
        """
        ...
```

### 5.3 Catalog Document Models

The typed output of normalization — what the load layer writes to MongoDB.

```python
@dataclass
class CatalogDistribution:
    """
    A fully normalized DataDistribution document ready for MongoDB.
    Fields map directly to the DataDistribution collection schema.
    """
    # Catalog model identity
    jrn:          str
    name:         str
    display_name: str           # = name (confirmed)
    description:  Optional[str]

    # Provenance block (source.*)
    provenance:   SourceProvenance

    # Connection context
    environment:      str       # PROD / UAT / DEV
    connection_name:  str       # workspace name
    database_name:    str       # UC schema name
    catalog_name:     Optional[str]

    # Managed Item base (lifecycle — partial; remaining fields set by catalog platform)
    lifecycle_status: str = "DRAFT"

    # Extended (populated beyond MVP)
    format:           Optional[str]         = None
    view_definition:  Optional[str]         = None
    owner:            Optional[str]         = None


@dataclass
class CatalogAttribute:
    """
    A fully normalized DataAttribute document ready for MongoDB.
    """
    jrn:              str
    parent_jrn:       str       # parent DataDistribution JRN
    name:             str
    display_name:     str       # = name
    description:      Optional[str]

    # Provenance
    provenance:       SourceProvenance

    # Connection context
    environment:      str
    connection_name:  str
    database_name:    str
    catalog_name:     Optional[str]
    table_name:       str

    # Column metadata
    data_type:        DataTypeEnum   # normalized
    raw_data_type:    str            # preserved for debugging

    # Column characteristics
    ordinal_position:    Optional[int]
    is_nullable:         Optional[bool]
    is_primary_key:      Optional[bool]
    is_foreign_key:      Optional[bool]
    is_unique:           Optional[bool]
    is_auto_incremented: Optional[bool]
    is_generated:        Optional[bool]
    precision:           Optional[int]
    scale:               Optional[int]
    max_length:          Optional[int]

    lifecycle_status: str = "DRAFT"


@dataclass
class CatalogLineageEvent:
    """
    A normalized lineage event for load into the lineage entity family.
    """
    event_id:         str
    event_timestamp:  datetime
    platform:         Platform

    source_jrn:       str            # resolved from source FQN
    target_jrn:       str            # resolved from target FQN
    source_col_jrn:   Optional[str]
    target_col_jrn:   Optional[str]

    producing_entity_type:  Optional[str]
    producing_entity_id:    Optional[str]
    producing_entity_name:  Optional[str]

    is_column_level:  bool
```

---

## 6. Load Layer Contract

The load layer receives `Catalog*` documents and writes them to MongoDB. It owns dual-write, upsert, soft-delete, and history emission.

```python
@runtime_checkable
class CatalogLoader(Protocol):
    """
    Writes normalized catalog documents to MongoDB.
    All writes are upserts keyed on `jrn`.
    """

    def upsert_distribution(
        self, doc: CatalogDistribution
    ) -> "LoadResult":
        """
        Upsert a DataDistribution document.
        - Keyed on `jrn`.
        - Dual-writes `source.*` AND `atlan.*` during the transition window.
        - Emits a DataDistributionHistory document if the document changed.
        """
        ...

    def upsert_attribute(
        self, doc: CatalogAttribute
    ) -> "LoadResult":
        """
        Upsert a DataAttribute document.
        - Keyed on `jrn`.
        - Dual-writes during transition window.
        - Emits DataAttributeHistory on change.
        """
        ...

    def soft_delete_distribution(self, jrn: str) -> "LoadResult":
        """
        Set source.sourceStatus = DELETED on a DataDistribution document.
        Also soft-deletes all child DataAttribute documents for this jrn.
        Never removes the MongoDB document.
        """
        ...

    def soft_delete_attribute(self, jrn: str) -> "LoadResult":
        """Set source.sourceStatus = DELETED on a DataAttribute document."""
        ...

    def upsert_lineage_event(
        self, event: CatalogLineageEvent
    ) -> "LoadResult":
        """
        Upsert a lineage event into the appropriate event collection.
        Idempotent on event_id.
        """
        ...


@dataclass
class LoadResult:
    jrn:        str
    operation:  str          # "inserted" | "updated" | "deleted" | "no_change"
    success:    bool
    error:      Optional[str] = None
```

---

## 7. Delta Engine Contract

The delta engine compares inventory snapshots to determine what changed. Produces the work list for detail extraction and the soft-delete list.

```python
@dataclass
class DeltaSet:
    """Output of the delta engine — the work list for one crawl run."""
    new_fqns:     list[str]   # in inventory, not in catalog — extract detail + load
    updated_fqns: list[str]   # in both, last_modified changed — extract detail + load
    deleted_fqns: list[str]   # in catalog, not in inventory — soft-delete


@runtime_checkable
class DeltaEngine(Protocol):

    def compute_distribution_delta(
        self,
        inventory: list[RawDistributionSummary],
        catalog_snapshot: list[dict],        # current catalog documents (jrn + last known FQN + modified)
        watermark: Optional[datetime],
    ) -> DeltaSet:
        """
        Compares the live inventory against what the catalog holds.
        - new_fqns:     in inventory but not in catalog snapshot
        - updated_fqns: in both, with last_modified > watermark
        - deleted_fqns: in catalog snapshot but not in inventory
        """
        ...

    def compute_attribute_delta(
        self,
        inventory: list[RawAttributeSummary],
        catalog_snapshot: list[dict],
        watermark: Optional[datetime],
    ) -> DeltaSet:
        ...
```

---

## 8. Pipeline Orchestration Contract

The pipeline wires all components together per run.

```python
@dataclass
class CrawlRun:
    """Metadata for one scheduled crawl execution."""
    run_id:       str
    platform:     Platform
    workspace:    str
    started_at:   datetime
    watermark:    Optional[datetime]   # last successful run's watermark


@dataclass
class CrawlRunResult:
    run_id:         str
    platform:       Platform
    workspace:      str
    started_at:     datetime
    completed_at:   datetime
    distributions_new:      int
    distributions_updated:  int
    distributions_deleted:  int
    attributes_new:         int
    attributes_updated:     int
    attributes_deleted:     int
    lineage_events:         int
    errors:                 list[str]
    success:                bool
    new_watermark:          Optional[datetime]


@runtime_checkable
class CrawlPipeline(Protocol):
    """
    Orchestrates one complete crawl run for one workspace.
    Sequence: connect → inventory → delta → detail extract → normalize → load → disconnect.
    """

    def run(self, crawl_run: CrawlRun) -> CrawlRunResult:
        """
        Execute a full crawl run.
        - Must be safe to re-run (idempotent).
        - A workspace-level failure must not propagate to other workspaces.
        - Persists the new watermark only on full success.
        """
        ...
```

---

## 9. Implementing a New Platform Connector

To add Snowflake (or any future platform), implement:

1. A concrete class implementing `MetadataConnector`.
2. A concrete class implementing `Normalizer` (or extend a shared base — platform-specific only in the raw→catalog mapping).
3. A concrete `DataTypeMapper` instance covering the new platform's type vocabulary.
4. Register the connector in the platform factory (not defined here — implementation detail).

The load layer, delta engine, and pipeline require no changes. The provenance block, JRN minting calls, environment lookup, and history emission are shared automatically.

### Minimum test surface for a new connector

- `get_distribution_inventory()` covers the entire in-scope estate without missing objects.
- `get_distribution_detail()` for a known table returns all expected attributes.
- `normalize_distribution()` produces a `CatalogDistribution` with a valid, stable `jrn` on two calls for the same input.
- `compute_distribution_delta()` correctly identifies all three change classes (new/updated/deleted) against a synthetic catalog snapshot.
- Soft-delete: a distribution removed from inventory produces `source.sourceStatus = DELETED` in the catalog, with no hard delete.

---

## 10. Interface Versioning & Evolution

| Change type | Classification | Required action |
|---|---|---|
| New optional field on a `Raw*` model | Non-breaking | Bumping patch version; normalizer ignores unknown fields |
| New required field on a `Raw*` model | **Breaking** | Minor version bump; all connectors updated before release |
| New method on `MetadataConnector` | **Breaking** | Minor version bump; all connectors implement before release |
| New enum value on `DataTypeEnum` or `Platform` | Non-breaking additive | Patch version; mapper updated |
| Rename / remove field or method | **Breaking** | Major version bump; migration plan required |

Current interface version: **`1.0.0`** (initial).
