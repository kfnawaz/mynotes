# S10b — Alteryx One Connector: Implementation Guide

**Status:** Deferred — gated on Alteryx One upgrade (May 2027 earliest)  
**Depends on:** S10a (OpenLineage receiver) complete, S10c (sidecar) deployed  
**Effort:** XL (~3–4 sprints, single developer)  
**Decision gate:** Alteryx One API must expose structured lineage — if it doesn't, S10c covers everything and S10b is unnecessary

---

## 1. Context

### 1.1 Why S10b Exists

JPMC is currently on Alteryx Server 2025.1. An upgrade to Alteryx 2025.2+ / Alteryx One is expected by May 2027. Alteryx One is a cloud-native redesign with a modern REST API that is expected to expose structured lineage — which the current Gallery API v1 does not.

S10c (the OpenLineage sidecar) covers both lineage and workflow metadata today using the Gallery API v1. S10b adds a **pull-based connector** that queries the Alteryx One API directly — same pattern as the Databricks, Snowflake, Tableau, and Oracle connectors.

### 1.2 What S10b Adds Over S10c

| Capability | S10c (sidecar — today) | S10b (connector — post upgrade) |
|---|---|---|
| Workflow metadata as catalog assets | ✅ via metadata sweep events | ✅ via MetadataConnector Protocol |
| Lineage edges | ✅ via Gallery API job polling | ✅ via native lineage API (if available) |
| Column-level lineage | ❌ | Possibly — depends on Alteryx One API |
| Workflow versioning | ❌ | ✅ version history from API |
| Schedule details | ⚠️ Limited | ✅ full schedule from API |
| Inter-workflow dependencies | ❌ | ✅ if API exposes them |
| Consistent with other connectors | ❌ push model | ✅ same pull model, same Protocol |

### 1.3 Decision Gate — Run Before Building

After the Alteryx One upgrade, run this check before starting S10b:

```
Does Alteryx One have a native lineage API returning
structured input/output connections?

  YES → Build S10b. Stop reading — go to section 3.
  
  NO  → S10c already covers everything.
        S10b is unnecessary. Close this story.
        Expand S10c if any gaps are found.
```

---

## 2. Architecture

### 2.1 Where S10b Fits

```
Alteryx One (post upgrade)
       │
       ├──► AlteryxConnector.get_distribution_inventory()    ← S10b (pull)
       │         → workflow metadata → DataDistribution
       │
       ├──► AlteryxLineageMiner.get_lineage_events()         ← S10b (pull)
       │         → native lineage API → DataLineageEvent
       │
       └──► Sidecar → OpenLineage Receiver                   ← S10c (push)
                 → backwards compatible, continues running
```

Both S10b and S10c can coexist. S10b provides scheduled full-inventory crawls. S10c provides near-real-time lineage events. They write to the same MongoDB collections — idempotent upserts keyed on JRN prevent duplicates.

### 2.2 JRN Format

```
Distribution:  jrn:jpm:ctcatalog::seal:{seal_node_id}::data-asset/distribution:alteryx/{server}/{workflow_name}
Attribute:     jrn:jpm:ctcatalog::seal:{seal_node_id}::data-attribute:alteryx/{server}/{workflow_name}/{tool_name}
```

All path components lowercased. `seal_node_id` resolved via `catalog_seal_map` keyed on server hostname.

### 2.3 Swappability Invariant

Adding Alteryx must require **zero changes** to existing layers — `Raw*` models, `Normalizer` Protocol, `CatalogLoader`, `DeltaEngine`, pipeline orchestration. If any downstream change is needed, the architecture leaked.

---

## 3. File Structure

```
src/datacompass_metadata_harvester/
    connectors/
        alteryx/                              ← NEW directory
            __init__.py
            auth.py                           ← Alteryx One OAuth + Gallery v1 key auth
            connector.py                      ← AlteryxConnector (MetadataConnector Protocol)
            queries.py                        ← API endpoints, URL builders, request helpers
            mapper.py                         ← API response → Raw* models
    normalization/
        alteryx_normalizer.py                 ← NEW — Raw* → Catalog*, JRN minting
        alteryx_type_mapper.py                ← NEW — tool types → DataTypeEnum
    lineage/
        alteryx_miner.py                      ← NEW — polls Alteryx One lineage API
    pipeline/
        alteryx_pipeline.py                   ← NEW — crawl + lineage pipelines
    config/
        loader.py                             ← EXTEND — AlteryxInstanceConfig + parser

scripts/
    run_crawl.py                              ← EXTEND — add alteryx to --platform choices
    run_lineage.py                            ← EXTEND — add alteryx to --platform choices

k8s/
    cronjobs/
        alteryx-crawler.yaml                  ← NEW
        alteryx-miner.yaml                    ← NEW

tests/
    connectors/alteryx/
        __init__.py
        test_auth.py
        test_connector.py
        test_mapper.py
    normalization/
        test_alteryx_normalizer.py
        test_alteryx_type_mapper.py
    lineage/
        test_alteryx_miner.py
        test_alteryx_miner_integration.py
    pipeline/
        test_alteryx_pipeline.py
```

---

## 4. Component Specifications

### 4.1 Auth — `connectors/alteryx/auth.py`

Two auth methods, selected via `auth_method` in config:

| `auth_method` | Protocol | Credential source | When to use |
|---|---|---|---|
| `alteryx_one_oauth` | OAuth2 client credentials | K8s Secret `alteryx-one-credentials` | Alteryx One (post upgrade) |
| `alteryx_gallery_key` | Gallery API v1 key/secret | K8s Secret `alteryx-gallery-credentials` | Alteryx Server 2025.1 (fallback) |

```python
from __future__ import annotations

import time
import threading
from dataclasses import dataclass, field
from datetime import datetime, UTC
from typing import Protocol

import requests


class AlteryxAuthError(Exception):
    """Raised when Alteryx authentication fails."""


class TokenProvider(Protocol):
    """Provides an auth token for Alteryx API calls."""
    def get_token(self) -> str: ...


@dataclass
class AlteryxOneOAuthProvider:
    """OAuth2 client_credentials grant for Alteryx One API.

    Token lifecycle:
        1. POST to token_url with client_id + client_secret
        2. Receive access_token + expires_in
        3. Cache token until 60 seconds before expiry
        4. Thread-safe — multiple pipeline threads can share one provider
    """
    client_id: str
    client_secret: str = field(repr=False)  # never in logs/tracebacks
    token_url: str  # e.g. https://alteryx-one.jpmchase.com/oauth2/token
    _cached_token: str = field(default="", repr=False, init=False)
    _expiry: float = field(default=0.0, init=False)
    _lock: threading.Lock = field(default_factory=threading.Lock, init=False)

    def get_token(self) -> str:
        """Return a valid access token, refreshing if needed."""
        with self._lock:
            if self._cached_token and time.time() < self._expiry:
                return self._cached_token
            return self._refresh()

    def _refresh(self) -> str:
        """POST to token endpoint. Cache result."""
        resp = requests.post(
            self.token_url,
            data={
                "grant_type": "client_credentials",
                "client_id": self.client_id,
                "client_secret": self.client_secret,
            },
            timeout=30,
        )
        if resp.status_code != 200:
            raise AlteryxAuthError(
                f"OAuth token request failed: {resp.status_code} {resp.text[:200]}"
            )
        data = resp.json()
        self._cached_token = data["access_token"]
        expires_in = int(data.get("expires_in", 3600))
        self._expiry = time.time() + expires_in - 60  # refresh 60s before expiry
        return self._cached_token


@dataclass
class GalleryV1AuthProvider:
    """Gallery API v1 auth — apiKey + apiSecret → OAuth token.

    Alteryx Server 2025.1 compatible. Used as fallback when
    Alteryx One is not yet available or for legacy servers.
    """
    server_url: str
    api_key: str
    api_secret: str = field(repr=False)
    _cached_token: str = field(default="", repr=False, init=False)
    _expiry: float = field(default=0.0, init=False)
    _lock: threading.Lock = field(default_factory=threading.Lock, init=False)

    def get_token(self) -> str:
        with self._lock:
            if self._cached_token and time.time() < self._expiry:
                return self._cached_token
            return self._refresh()

    def _refresh(self) -> str:
        resp = requests.post(
            f"{self.server_url.rstrip('/')}/gallery/v1/oauth2/token",
            data={
                "grant_type": "client_credentials",
                "client_id": self.api_key,
                "client_secret": self.api_secret,
            },
            timeout=30,
        )
        if resp.status_code != 200:
            raise AlteryxAuthError(
                f"Gallery token request failed: {resp.status_code}"
            )
        data = resp.json()
        self._cached_token = data["access_token"]
        self._expiry = time.time() + int(data.get("expires_in", 3600)) - 60
        return self._cached_token


def make_token_provider(
    auth_method: str,
    server_url: str,
    secret_dir: str,
) -> TokenProvider:
    """Factory — selects auth provider based on config.

    Reads credentials from K8s Secret files mounted at secret_dir:
        alteryx_one_oauth:   client_id, client_secret, token_url
        alteryx_gallery_key: api_key, api_secret
    Falls back to environment variables for dev/testing.
    """
    ...
```

---

### 4.2 Queries — `connectors/alteryx/queries.py`

API endpoint definitions. Two API versions supported — the connector selects based on `api_version` config.

```python
"""Alteryx API endpoint definitions and request builders.

Two API versions:
    v3 — Alteryx One (post upgrade). Full REST API with lineage.
    v1 — Gallery API (current Alteryx Server 2025.1). Limited REST.

The connector selects endpoints based on AlteryxInstanceConfig.api_version.
"""
from __future__ import annotations
from dataclasses import dataclass


# ──────────────────────────────────────────────
# Alteryx One API v3 (expected — exact paths TBD)
# ──────────────────────────────────────────────
V3_WORKFLOWS_LIST = "/v3/workflows"
V3_WORKFLOW_DETAIL = "/v3/workflows/{workflow_id}"
V3_WORKFLOW_TOOLS = "/v3/workflows/{workflow_id}/tools"
V3_WORKFLOW_RUNS = "/v3/workflows/{workflow_id}/runs"
V3_WORKFLOW_LINEAGE = "/v3/workflows/{workflow_id}/lineage"
V3_PROJECTS_LIST = "/v3/projects"

# ──────────────────────────────────────────────
# Gallery API v1 (known — works with 2025.1)
# ──────────────────────────────────────────────
V1_APPS_LIST = "/gallery/v1/apps"
V1_APP_DETAIL = "/gallery/v1/apps/{app_id}"
V1_JOBS_LIST = "/gallery/v1/jobs/all"


@dataclass(frozen=True)
class APIEndpoints:
    """Resolved endpoint URLs for a specific API version."""
    workflows_list: str
    workflow_detail: str
    workflow_tools: str | None  # v1 doesn't have this
    workflow_runs: str | None   # v1 doesn't have this
    workflow_lineage: str | None  # v1 doesn't have this — key differentiator
    projects_list: str | None


def endpoints_for_version(api_version: str) -> APIEndpoints:
    """Return the right endpoints for the configured API version."""
    if api_version == "v3":
        return APIEndpoints(
            workflows_list=V3_WORKFLOWS_LIST,
            workflow_detail=V3_WORKFLOW_DETAIL,
            workflow_tools=V3_WORKFLOW_TOOLS,
            workflow_runs=V3_WORKFLOW_RUNS,
            workflow_lineage=V3_WORKFLOW_LINEAGE,
            projects_list=V3_PROJECTS_LIST,
        )
    else:  # v1 fallback
        return APIEndpoints(
            workflows_list=V1_APPS_LIST,
            workflow_detail=V1_APP_DETAIL,
            workflow_tools=None,
            workflow_runs=None,
            workflow_lineage=None,  # not available in v1
            projects_list=None,
        )
```

---

### 4.3 Connector — `connectors/alteryx/connector.py`

Full `MetadataConnector` Protocol implementation.

```python
"""Alteryx connector — polls Alteryx One or Gallery API for workflow metadata.

Workflows map to DataDistribution documents.
Workflow tools/steps optionally map to DataAttribute documents.

Auth method selected via config.auth_method:
    alteryx_one_oauth  — Alteryx One OAuth2 (post upgrade)
    alteryx_gallery_key — Gallery API v1 key/secret (current)

API version selected via config.api_version:
    v3 — Alteryx One (full lineage, tools, runs)
    v1 — Gallery (workflow list + detail only)
"""
from __future__ import annotations

from collections.abc import Iterator
from dataclasses import dataclass, field
from datetime import datetime
from typing import Any

import requests

from datacompass_metadata_harvester.connectors.alteryx.auth import (
    TokenProvider,
    make_token_provider,
)
from datacompass_metadata_harvester.connectors.alteryx.queries import (
    APIEndpoints,
    endpoints_for_version,
)
from datacompass_metadata_harvester.connectors.alteryx.mapper import (
    map_workflow_to_distribution_summary,
    map_workflow_to_distribution,
    map_tool_to_attribute,
)
from datacompass_metadata_harvester.models.shared import (
    CrawlScope,
    ExtractionResult,
    Platform,
    ScopeFilter,
)
from datacompass_metadata_harvester.models.raw import (
    RawAttribute,
    RawDistribution,
    RawDistributionSummary,
)
from datacompass_metadata_harvester.observability.logging import get_logger

_log = get_logger(__name__)


@dataclass
class AlteryxConnector:
    """Alteryx MetadataConnector implementation.

    Implements the same Protocol as DatabricksConnector, SnowflakeConnector,
    TableauConnector, and OracleConnector. The pipeline, normalizer, delta
    engine, and load layer never know which connector they're talking to.
    """
    _server_url: str
    _token_provider: TokenProvider
    _endpoints: APIEndpoints
    _scope: CrawlScope
    _session: requests.Session | None = field(default=None, init=False)

    def connect(self) -> None:
        """Acquire auth token and open HTTP session."""
        self._session = requests.Session()
        token = self._token_provider.get_token()
        self._session.headers.update({
            "Authorization": f"Bearer {token}",
            "Accept": "application/json",
        })
        # Verify connectivity
        resp = self._session.get(
            f"{self._server_url}{self._endpoints.workflows_list}",
            params={"limit": 1},
            timeout=30,
        )
        resp.raise_for_status()
        _log.info("alteryx_connected", server=self._server_url)

    def disconnect(self) -> None:
        """Close HTTP session."""
        if self._session:
            self._session.close()
            self._session = None
        _log.info("alteryx_disconnected")

    def get_distribution_inventory(
        self,
    ) -> Iterator[ExtractionResult[RawDistributionSummary]]:
        """List all workflows visible to the service account.

        Scope filtering applied:
            include/exclude_projects — glob match on project name
            include/exclude_datasources — glob match on workflow name
              (reusing 'datasources' scope field since workflows are
               the primary distribution unit in Alteryx, analogous to
               Tableau datasources)

        Pagination: follows API next_page cursor if present.
        """
        assert self._session is not None
        url = f"{self._server_url}{self._endpoints.workflows_list}"
        page = 1

        while url:
            resp = self._session.get(url, timeout=60)
            resp.raise_for_status()
            data = resp.json()

            workflows = data if isinstance(data, list) else data.get("results", [])

            for wf in workflows:
                project = wf.get("project", "")
                name = wf.get("name", "")

                # Scope filter — project
                if not ScopeFilter.is_included(
                    project,
                    self._scope.include_projects,
                    self._scope.exclude_projects,
                ):
                    continue

                # Scope filter — workflow name (using datasources field)
                if not ScopeFilter.is_included(
                    name,
                    self._scope.include_datasources,
                    self._scope.exclude_datasources,
                ):
                    continue

                try:
                    summary = map_workflow_to_distribution_summary(wf)
                    yield ExtractionResult(record=summary)
                except Exception as exc:
                    yield ExtractionResult(
                        error=f"workflow_map_error [{name}]: {exc}"
                    )

            # Pagination — follow next link if API provides one
            url = None
            if isinstance(data, dict):
                next_link = data.get("next", data.get("nextPage"))
                if next_link:
                    url = (
                        next_link
                        if next_link.startswith("http")
                        else f"{self._server_url}{next_link}"
                    )

    def get_distribution_detail(
        self, fqn: str,
    ) -> ExtractionResult[RawDistribution]:
        """Fetch full metadata for a specific workflow by ID.

        fqn format: "{server_hostname}/{workflow_id}"
        The workflow_id is extracted from the fqn to build the API URL.
        """
        assert self._session is not None
        workflow_id = fqn.split("/")[-1]
        url = f"{self._server_url}{self._endpoints.workflow_detail}".format(
            workflow_id=workflow_id,
        )
        try:
            resp = self._session.get(url, timeout=60)
            resp.raise_for_status()
            wf = resp.json()
            dist = map_workflow_to_distribution(wf)
            return ExtractionResult(record=dist)
        except Exception as exc:
            return ExtractionResult(
                error=f"workflow_detail_error [{fqn}]: {exc}"
            )

    def get_attribute_details(
        self, fqn: str,
    ) -> Iterator[ExtractionResult[RawAttribute]]:
        """Fetch workflow tool/step details (v3 API only).

        Each tool in the workflow becomes a RawAttribute:
            Input tools (database readers) → connection metadata
            Output tools (database writers) → connection metadata
            Transform tools → tool type and config summary

        Returns empty iterator if tools endpoint is not available (v1 API).
        """
        if self._endpoints.workflow_tools is None:
            return  # v1 API — tools not available

        assert self._session is not None
        workflow_id = fqn.split("/")[-1]
        url = f"{self._server_url}{self._endpoints.workflow_tools}".format(
            workflow_id=workflow_id,
        )
        try:
            resp = self._session.get(url, timeout=60)
            resp.raise_for_status()
            tools = resp.json()
            if isinstance(tools, dict):
                tools = tools.get("results", [])

            for ordinal, tool in enumerate(tools):
                try:
                    attr = map_tool_to_attribute(tool, fqn, ordinal)
                    yield ExtractionResult(record=attr)
                except Exception as exc:
                    yield ExtractionResult(
                        error=f"tool_map_error [{fqn}/{tool.get('name', '?')}]: {exc}"
                    )
        except Exception as exc:
            yield ExtractionResult(
                error=f"tools_fetch_error [{fqn}]: {exc}"
            )

    @property
    def platform(self) -> Platform:
        return Platform.ALTERYX

    @classmethod
    def from_config(cls, config: Any) -> AlteryxConnector:
        """Factory — builds connector from AlteryxInstanceConfig."""
        token_provider = make_token_provider(
            auth_method=config.auth_method,
            server_url=config.server_url,
            secret_dir=f"/var/run/secrets/{config.secret_name}",
        )
        endpoints = endpoints_for_version(config.api_version)
        return cls(
            _server_url=config.server_url.rstrip("/"),
            _token_provider=token_provider,
            _endpoints=endpoints,
            _scope=config.scope,
        )
```

---

### 4.4 Mapper — `connectors/alteryx/mapper.py`

```python
"""Map Alteryx API responses to Raw* domain models.

Two API versions produce different JSON shapes — the mapper handles both.
The caller tells the mapper which version by calling the right function.
All functions produce identical Raw* objects regardless of API version.
"""
from __future__ import annotations

from datetime import datetime, UTC
from typing import Any

from datacompass_metadata_harvester.models.raw import (
    RawAttribute,
    RawDistribution,
    RawDistributionSummary,
)


def map_workflow_to_distribution_summary(wf: dict[str, Any]) -> RawDistributionSummary:
    """Map an Alteryx workflow list item to RawDistributionSummary.

    Works with both v3 API and Gallery v1 API response shapes.

    v3 fields:    id, name, project.name, owner.email, updatedAt
    v1 fields:    id, name, project,      owner,       lastModifiedDate

    Returns a RawDistributionSummary with:
        platform_id:   workflow id
        catalog:       server hostname (from config, not from API)
        schema:        project name
        table:         workflow name (lowercase)
        table_type:    "WORKFLOW"
        last_modified: updatedAt or lastModifiedDate
    """
    ...


def map_workflow_to_distribution(wf: dict[str, Any]) -> RawDistribution:
    """Map full workflow detail to RawDistribution.

    Additional fields over summary:
        description, schedule, version, run_count,
        create_date, tags, is_public

    All stored in RawDistribution.properties dict for
    normalization layer to extract what it needs.
    """
    ...


def map_tool_to_attribute(
    tool: dict[str, Any],
    workflow_fqn: str,
    ordinal: int,
) -> RawAttribute:
    """Map a workflow tool/step to RawAttribute (v3 API only).

    Tool types and their mapping:
        AlteryxInput / DbFileInput   → "DATABASE_INPUT"
        AlteryxOutput / DbFileOutput → "DATABASE_OUTPUT"
        Join                         → "TRANSFORM_JOIN"
        Filter                       → "TRANSFORM_FILTER"
        Formula                      → "TRANSFORM_FORMULA"
        Summarize                    → "TRANSFORM_SUMMARIZE"
        Union                        → "TRANSFORM_UNION"
        Sort                         → "TRANSFORM_SORT"
        Select                       → "TRANSFORM_SELECT"
        (all others)                 → "TOOL"

    Returns a RawAttribute with:
        parent_fqn:    workflow FQN
        column_name:   tool name (lowercase)
        ordinal:       tool position in workflow
        raw_data_type: tool type string from above
        description:   tool annotation/comment if present
    """
    ...
```

---

### 4.5 Normalizer — `normalization/alteryx_normalizer.py`

```python
"""Normalize Alteryx Raw* models to Catalog* documents.

JRN format:
    Distribution: jrn:jpm:ctcatalog::seal:{seal_node_id}::
                  data-asset/distribution:alteryx/{server}/{workflow_name}
    Attribute:    jrn:jpm:ctcatalog::seal:{seal_node_id}::
                  data-attribute:alteryx/{server}/{workflow_name}/{tool_name}

All path components lowercased.
seal_node_id resolved via catalog_seal_map keyed on server hostname.
"""
from __future__ import annotations

from datacompass_metadata_harvester.models.catalog import (
    CatalogAttribute,
    CatalogDistribution,
)
from datacompass_metadata_harvester.models.raw import RawAttribute, RawDistribution
from datacompass_metadata_harvester.models.shared import AssetStatus, Platform
from datacompass_metadata_harvester.normalization.jrn import JrnBuilder
from datacompass_metadata_harvester.normalization.registry import WorkspaceInstance


class AlteryxNormalizer:
    """Normalizes Alteryx metadata to catalog format.

    Same interface as DatabricksNormalizer, SnowflakeNormalizer,
    TableauNormalizer, OracleNormalizer.
    """

    def __init__(self, workspace: WorkspaceInstance) -> None:
        self._workspace = workspace

    def normalize_distribution(
        self,
        raw: RawDistribution,
        status: AssetStatus,
    ) -> CatalogDistribution:
        """Convert RawDistribution (workflow) to CatalogDistribution.

        JRN path: alteryx/{server_hostname}/{workflow_name}
        seal_node_id: resolved from catalog_seal_map using server hostname
        platform: Platform.ALTERYX
        type_name: "WORKFLOW"
        """
        seal_node_id = self._workspace.resolve_seal_node_id(raw.catalog)
        jrn = JrnBuilder.mint_distribution(
            seal_node_id=seal_node_id,
            platform="alteryx",
            path_parts=[raw.catalog, raw.table],  # server/workflow_name
        )
        return CatalogDistribution(
            jrn=jrn,
            name=raw.table,
            platform=Platform.ALTERYX,
            environment=self._workspace.environment,
            status=status,
            type_name=raw.table_type,  # "WORKFLOW"
            qualified_name=f"{raw.catalog}.{raw.schema}.{raw.table}",
            catalog=raw.catalog,
            schema=raw.schema,
            table=raw.table,
            last_modified=raw.last_modified,
            properties=raw.properties,
        )

    def normalize_attribute(
        self,
        raw: RawAttribute,
        parent_jrn: str,
    ) -> CatalogAttribute:
        """Convert RawAttribute (workflow tool) to CatalogAttribute.

        JRN path: alteryx/{server_hostname}/{workflow_name}/{tool_name}
        """
        seal_node_id = self._workspace.resolve_seal_node_id(raw.catalog)
        jrn = JrnBuilder.mint_attribute(
            seal_node_id=seal_node_id,
            platform="alteryx",
            path_parts=[raw.catalog, raw.parent_table, raw.column_name],
        )
        return CatalogAttribute(
            jrn=jrn,
            parent_jrn=parent_jrn,
            name=raw.column_name,
            platform=Platform.ALTERYX,
            ordinal=raw.ordinal,
            raw_data_type=raw.raw_data_type,
            description=raw.description,
        )
```

---

### 4.6 Type Mapper — `normalization/alteryx_type_mapper.py`

```python
"""Map Alteryx tool types to DataTypeEnum.

Alteryx tools are processing steps, not column types — they don't map
cleanly to SQL data types. Most map to OTHER. The real type information
lives in the database connections that Alteryx reads from and writes to,
which are already cataloged as Databricks/Snowflake/Oracle tables with
proper column types.

This mapper exists for completeness — the DataAttribute for a workflow
tool records what kind of tool it is, not what data type it produces.
"""
from datacompass_metadata_harvester.models.shared import DataTypeEnum

_ALTERYX_TOOL_TYPE_MAP: dict[str, DataTypeEnum] = {
    "DATABASE_INPUT": DataTypeEnum.OTHER,
    "DATABASE_OUTPUT": DataTypeEnum.OTHER,
    "TRANSFORM_JOIN": DataTypeEnum.OTHER,
    "TRANSFORM_FILTER": DataTypeEnum.OTHER,
    "TRANSFORM_FORMULA": DataTypeEnum.OTHER,
    "TRANSFORM_SUMMARIZE": DataTypeEnum.OTHER,
    "TRANSFORM_UNION": DataTypeEnum.OTHER,
    "TRANSFORM_SORT": DataTypeEnum.OTHER,
    "TRANSFORM_SELECT": DataTypeEnum.OTHER,
    "TOOL": DataTypeEnum.OTHER,
}


def map_alteryx_type(raw_type: str) -> DataTypeEnum:
    """Map an Alteryx tool type string to DataTypeEnum.

    Returns DataTypeEnum.OTHER for all types — Alteryx tools are
    processing steps, not data types.
    """
    return _ALTERYX_TOOL_TYPE_MAP.get(raw_type.upper(), DataTypeEnum.OTHER)
```

---

### 4.7 Lineage Miner — `lineage/alteryx_miner.py`

```python
"""Alteryx lineage miner — polls Alteryx One's native lineage API.

KEY DIFFERENCE from S9b/S9c miners:
    Snowflake/Databricks miners parse raw SQL from query history using sqlglot.
    This miner reads pre-computed structured lineage from the Alteryx One API.
    No raw SQL is involved — no redaction needed.

The lineage API (v3 only) returns structured input/output connections:
    {
        "inputs": [
            {"connectionType": "ODBC", "server": "...", "database": "...",
             "schema": "...", "table": "..."}
        ],
        "outputs": [...]
    }

These are resolved to JRNs using the same ConnectionRegistry used by
the S10a OpenLineage receiver. This ensures consistent JRN resolution
regardless of whether lineage came from the sidecar (push) or the
connector (pull).

Falls back gracefully when lineage API is unavailable (v1 API):
    - Logs a warning
    - Yields nothing
    - Does not fail the pipeline
"""
from __future__ import annotations

from collections.abc import Iterator
from dataclasses import dataclass, field
from datetime import datetime, UTC
from typing import Any

import requests

from datacompass_metadata_harvester.connectors.alteryx.auth import TokenProvider
from datacompass_metadata_harvester.connectors.alteryx.queries import APIEndpoints
from datacompass_metadata_harvester.lineage.connection_registry import (
    ConnectionRegistry,
)
from datacompass_metadata_harvester.models.raw import RawLineageEvent
from datacompass_metadata_harvester.models.shared import (
    ExtractionResult,
    LineageScope,
    Platform,
    ScopeFilter,
)
from datacompass_metadata_harvester.normalization.jrn import JrnBuilder
from datacompass_metadata_harvester.observability.logging import get_logger

_log = get_logger(__name__)


@dataclass
class AlteryxLineageMiner:
    """Mine lineage from Alteryx One's native lineage API.

    Two-step process per workflow:
        1. GET /v3/workflows/{id}/runs — find runs since watermark
        2. GET /v3/workflows/{id}/lineage — get structured connections

    For each (input, output) pair:
        1. Resolve input connection to a namespace string
        2. Resolve namespace to a JRN via ConnectionRegistry
        3. Same for output
        4. Yield RawLineageEvent

    Respects LineageScope:
        enabled: if False, yield nothing
        exclude_users: filter by workflow owner
        batch_size: limit workflows processed per run
        exclude_catalogs: filter out edges to/from excluded catalogs
    """
    _server_url: str
    _token_provider: TokenProvider
    _endpoints: APIEndpoints
    _connection_registry: ConnectionRegistry
    _lineage_scope: LineageScope
    _session: requests.Session | None = field(default=None, init=False)

    def connect(self) -> None:
        self._session = requests.Session()
        token = self._token_provider.get_token()
        self._session.headers.update({
            "Authorization": f"Bearer {token}",
            "Accept": "application/json",
        })

    def disconnect(self) -> None:
        if self._session:
            self._session.close()
            self._session = None

    def get_lineage_events(
        self,
        since: datetime,
        batch_size: int = 500,
    ) -> Iterator[ExtractionResult[RawLineageEvent]]:
        """Yield lineage events from Alteryx One API.

        If lineage endpoint is unavailable (v1 API), yields nothing.
        No raw SQL is involved — no redaction needed.
        """
        if not self._lineage_scope.enabled:
            _log.info("alteryx_lineage_disabled")
            return

        if self._endpoints.workflow_lineage is None:
            _log.warning(
                "alteryx_lineage_not_available",
                reason="v1 API does not expose lineage endpoint",
            )
            return

        assert self._session is not None
        effective_batch = self._lineage_scope.batch_size or batch_size

        # Step 1: Get workflows that ran since watermark
        # Step 2: For each, get lineage
        # Step 3: Resolve connections to JRNs
        # Step 4: Yield RawLineageEvent per edge
        ...
```

---

### 4.8 Config — `config/loader.py` Extension

```python
@dataclass
class AlteryxInstanceConfig:
    """Per-workspace configuration for Alteryx connector."""
    workspace_name: str
    catalog_seal_map: dict[str, str]
    environment: str
    server_url: str
    auth_method: str            # "alteryx_one_oauth" or "alteryx_gallery_key"
    secret_name: str
    api_version: str = "v3"     # "v3" for Alteryx One, "v1" for Gallery fallback
    default_seal_node_id: str | None = None
    scope: CrawlScope = field(default_factory=CrawlScope)
    lineage_scope: LineageScope = field(default_factory=LineageScope)

    def to_workspace_instance(self) -> WorkspaceInstance:
        return WorkspaceInstance(
            workspace_name=self.workspace_name,
            catalog_seal_map=self.catalog_seal_map,
            environment=self.environment,
            default_seal_node_id=self.default_seal_node_id,
        )

    def to_connector_config(self) -> ConnectorConfig:
        return ConnectorConfig(
            platform=Platform.ALTERYX,
            workspace_name=self.workspace_name,
            environment=self.environment,
            secret_name=self.secret_name,
            extra={
                "server_url": self.server_url,
                "auth_method": self.auth_method,
                "api_version": self.api_version,
            },
            scope=self.scope,
        )
```

```yaml
# config.yaml section
alteryx_workspaces:
  - workspace_name: alteryx-one-prod
    auth_method: alteryx_one_oauth
    environment: PROD
    server_url: https://alteryx-one.jpmchase.com
    api_version: v3                         # v3 for Alteryx One, v1 for Gallery
    secret_name: alteryx-one-credentials
    catalog_seal_map:
      alteryx-prod: "34243"
    default_seal_node_id: "34243"
    scope:
      exclude_projects:
        - "*_test"
        - "*_sandbox"
        - "*_dev"
    lineage_scope:
      enabled: true
      lookback_hours: 24
      batch_size: 500
      exclude_users:
        - system
        - service-account
```

---

### 4.9 Pipeline — `pipeline/alteryx_pipeline.py`

```python
"""Alteryx crawl and lineage pipelines.

Same pattern as all other platform pipelines:
    CrawlPipeline:   connect → inventory → delta → detail → normalize → load → watermark
    LineagePipeline:  connect → get_lineage_events → normalize → load → watermark

Uses crawler_watermarks for crawl, lineage_watermarks for lineage (separate collections).
Watermark persisted only on full success.
"""
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING

import pymongo

from datacompass_metadata_harvester.connectors.alteryx.connector import AlteryxConnector
from datacompass_metadata_harvester.delta.watermark import WatermarkStore
from datacompass_metadata_harvester.models.shared import Platform
from datacompass_metadata_harvester.observability.logging import get_logger

if TYPE_CHECKING:
    from datacompass_metadata_harvester.config.loader import AlteryxInstanceConfig

_log = get_logger(__name__)


class AlteryxCrawlPipeline:
    """Orchestrates one metadata crawl pass per Alteryx workspace.
    Same lifecycle as DatabricksCrawlPipeline.
    """

    @classmethod
    def from_config(
        cls,
        config: AlteryxInstanceConfig,
        client: pymongo.MongoClient,  # type: ignore[type-arg]
    ) -> AlteryxCrawlPipeline:
        ...

    def run(self) -> CrawlRunResult:
        ...


class AlteryxLineagePipeline:
    """Orchestrates one lineage mining pass per Alteryx workspace.
    Same lifecycle as SnowflakeLineagePipeline.
    Uses lineage_watermarks collection (not crawler_watermarks).
    """

    @classmethod
    def from_config(
        cls,
        config: AlteryxInstanceConfig,
        client: pymongo.MongoClient,  # type: ignore[type-arg]
    ) -> AlteryxLineagePipeline:
        ...

    def run(self) -> LineageRunResult:
        ...
```

---

## 5. K8s Manifests

### `k8s/cronjobs/alteryx-crawler.yaml`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: alteryx-crawler
  namespace: datacompass
  labels:
    app: datacompass-metadata-harvester
    component: crawler
    platform: alteryx
spec:
  schedule: "0 7 * * *"           # daily at 7am UTC
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 3
  successfulJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 1
      activeDeadlineSeconds: 3600  # 1 hour
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: crawler
              image: artifactory.jpmchase.com/docker/datacompass-metadata-harvester:IMAGE_TAG
              command: [poetry, run, python, scripts/run_crawl.py]
              args: [--platform, alteryx, --config, /config/config.yaml]
              env:
                - name: MONGODB_URI
                  valueFrom:
                    secretKeyRef:
                      name: mongodb-credentials
                      key: uri
              envFrom:
                - secretRef:
                    name: alteryx-one-credentials
              volumeMounts:
                - name: config
                  mountPath: /config
                  readOnly: true
          volumes:
            - name: config
              configMap:
                name: datacompass-config
```

### `k8s/cronjobs/alteryx-miner.yaml`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: alteryx-miner
  namespace: datacompass
  labels:
    app: datacompass-metadata-harvester
    component: miner
    platform: alteryx
spec:
  schedule: "15 */4 * * *"        # every 4 hours at :15
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 3
  successfulJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 1
      activeDeadlineSeconds: 1800  # 30 min
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: miner
              image: artifactory.jpmchase.com/docker/datacompass-metadata-harvester:IMAGE_TAG
              command: [poetry, run, python, scripts/run_lineage.py]
              args: [--platform, alteryx, --config, /config/config.yaml]
              env:
                - name: MONGODB_URI
                  valueFrom:
                    secretKeyRef:
                      name: mongodb-credentials
                      key: uri
              envFrom:
                - secretRef:
                    name: alteryx-one-credentials
              volumeMounts:
                - name: config
                  mountPath: /config
                  readOnly: true
          volumes:
            - name: config
              configMap:
                name: datacompass-config
```

---

## 6. Test Plan

### Unit Tests (mocked — no real Alteryx needed)

| Test file | Tests | What it covers |
|---|---|---|
| `tests/connectors/alteryx/test_auth.py` | 6 | OAuth token fetch, caching, expiry refresh, Gallery v1 auth, error handling |
| `tests/connectors/alteryx/test_connector.py` | 10 | Inventory pagination, scope filtering, detail fetch, tools fetch, v1 fallback, error handling |
| `tests/connectors/alteryx/test_mapper.py` | 8 | v3 response mapping, v1 response mapping, tool mapping, missing fields, edge cases |
| `tests/normalization/test_alteryx_normalizer.py` | 6 | JRN format, seal_node_id resolution, unmapped catalog failure, attribute normalization |
| `tests/normalization/test_alteryx_type_mapper.py` | 4 | All tool types → OTHER, unknown type → OTHER |
| `tests/lineage/test_alteryx_miner.py` | 10 | Lineage fetch, connection resolution, JRN minting, v1 fallback yields nothing, scope filtering, batch size |
| `tests/pipeline/test_alteryx_pipeline.py` | 8 | Crawl lifecycle, lineage lifecycle, watermark only on success, from_config factory |

**Total: ~52 unit tests**

### Integration Tests (needs real Alteryx One)

| Test file | Tests | What it covers |
|---|---|---|
| `tests/connectors/alteryx/test_connector_integration.py` | 2 | Real API connectivity, inventory returns workflows |
| `tests/lineage/test_alteryx_miner_integration.py` | 2 | Real lineage API, events written to MongoDB |

Auto-skip when `ALTERYX_SERVER_URL`, `ALTERYX_API_KEY`, `ALTERYX_API_SECRET`, `MONGODB_TEST_URI` env vars are absent.

---

## 7. Effort Summary

| Component | Effort | Days | Depends on |
|---|---|---|---|
| Auth (`auth.py`) | S | 1–2 | Alteryx One API docs |
| Queries (`queries.py`) | S | 1 | Alteryx One API docs |
| Connector (`connector.py`) | L | 5–7 | Auth, queries |
| Mapper (`mapper.py`) | M | 2–3 | Connector |
| Normalizer (`alteryx_normalizer.py`) | M | 2–3 | Mapper |
| Type mapper (`alteryx_type_mapper.py`) | S | 1 | — |
| Lineage miner (`alteryx_miner.py`) | L | 5–7 | Connector, ConnectionRegistry |
| Pipeline (`alteryx_pipeline.py`) | M | 2–3 | All above |
| Config loader extension | S | 1 | — |
| Scripts + K8s manifests | S | 1 | — |
| Unit tests (~52 tests) | L | 5–7 | All above |
| Integration tests | M | 3–4 | Real Alteryx One access |
| **Total** | **XL** | **~28–40 days** | |

**In sprints:** approximately 3–4 sprints (2-week sprints, single developer).

---

## 8. Claude Code Prompt — Build S10b

Paste this into Claude Code when the Alteryx One API is available and the decision gate in section 1.3 passes:

> Implement S10b — the Alteryx One connector and lineage miner. Follow the exact same patterns as S8 (Snowflake, Tableau) and S11 (Oracle). This is a new platform connector — no changes to existing connectors, normalizers, load layer, delta engine, or existing pipelines.
>
> **Reference the implementation guide at `docs/s10b_alteryx_one_connector.md`** — it contains the full file structure, all component specifications with code, mapper logic, normalizer JRN format, type mapper, lineage miner design, pipeline pattern, config loader extension, K8s manifests, and test plan.
>
> Key constraints:
> - Swappability invariant: zero changes to existing layers (`Raw*` models, `Normalizer` Protocol, `CatalogLoader`, `DeltaEngine`, pipeline base)
> - Auth: two methods via `auth_method` config — `alteryx_one_oauth` (OAuth2 client credentials) and `alteryx_gallery_key` (Gallery v1 fallback)
> - API version: two versions via `api_version` config — `v3` (Alteryx One, full lineage) and `v1` (Gallery, no lineage endpoint)
> - JRN path: `alteryx/{server_hostname}/{workflow_name}` — all components lowercased
> - Lineage: native API (v3), not SQL parsing — no redaction needed (unlike S9b/S9c)
> - Lineage miner: use existing `ConnectionRegistry` from `lineage/connection_registry.py` for namespace → JRN resolution (same registry as S10a OpenLineage receiver)
> - Scope filtering: `include/exclude_projects` for project-level, `include/exclude_datasources` for workflow-level (reusing datasources scope field)
> - `lineage_scope`: respect `enabled`, `batch_size`, `exclude_users`
> - v1 fallback: when `workflow_lineage` endpoint is None (v1 API), miner yields nothing gracefully — does not fail
>
> Implement all components from the guide:
> 1. `connectors/alteryx/auth.py` — `AlteryxOneOAuthProvider`, `GalleryV1AuthProvider`, `make_token_provider` factory
> 2. `connectors/alteryx/queries.py` — `APIEndpoints` dataclass, `endpoints_for_version()` factory
> 3. `connectors/alteryx/connector.py` — `AlteryxConnector` implementing `MetadataConnector` Protocol
> 4. `connectors/alteryx/mapper.py` — `map_workflow_to_distribution_summary`, `map_workflow_to_distribution`, `map_tool_to_attribute`
> 5. `normalization/alteryx_normalizer.py` — `AlteryxNormalizer`
> 6. `normalization/alteryx_type_mapper.py` — `map_alteryx_type()`
> 7. `lineage/alteryx_miner.py` — `AlteryxLineageMiner`
> 8. `pipeline/alteryx_pipeline.py` — `AlteryxCrawlPipeline`, `AlteryxLineagePipeline`
> 9. `config/loader.py` — `AlteryxInstanceConfig`, `_parse_alteryx_scope()`
> 10. `scripts/run_crawl.py` — add `alteryx` to `--platform` choices
> 11. `scripts/run_lineage.py` — add `alteryx` to `--platform` choices
> 12. `k8s/cronjobs/alteryx-crawler.yaml` — daily at 07:00, 1h deadline
> 13. `k8s/cronjobs/alteryx-miner.yaml` — every 4h at :15, 30min deadline
> 14. All unit tests from test plan (~52 tests)
> 15. Integration tests with auto-skip
>
> Run `poetry run pytest` and `poetry run mypy` to green before finishing. Do not use `--unsafe-fixes`.

---

## 9. Post-Implementation Verification

Run these checks after Claude Code finishes:

```bash
# 1. Swappability — no existing files changed
git diff HEAD --name-only | grep -v alteryx | grep -v run_crawl | grep -v run_lineage

# 2. No redaction needed — no raw SQL in miner
grep -n "redact_sql\|sql_text\|statement_text" src/datacompass_metadata_harvester/lineage/alteryx_miner.py

# 3. v1 fallback graceful — yields nothing, doesn't fail
grep -n "workflow_lineage is None\|not_available" src/datacompass_metadata_harvester/lineage/alteryx_miner.py

# 4. JRN format correct
grep -n "alteryx" src/datacompass_metadata_harvester/normalization/alteryx_normalizer.py

# 5. ConnectionRegistry reused
grep -n "ConnectionRegistry\|connection_registry" src/datacompass_metadata_harvester/lineage/alteryx_miner.py

# 6. All tests pass
poetry run pytest -q && poetry run mypy && poetry run ruff check .
```
