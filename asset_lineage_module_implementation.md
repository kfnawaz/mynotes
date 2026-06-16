# Asset Explorer & Lineage Module — Implementation Stages

**Reference:** `docs/asset_lineage_module_architecture.md`  
**Apps modified:** Workflow Management API (WMA), Control Plane SPA  
**Data source:** MongoDB collections written by the Harvester (read-only)

---

## Implementation Overview

Seven stages, each independently deployable and testable:

```
Stage 1 — API: Asset catalog + detail + attributes endpoints
Stage 2 — SPA: Asset catalog page (search + browse + filters)
Stage 3 — SPA: Asset detail page (overview + attributes tabs)
Stage 4 — API: Lineage graph assembly + impact analysis endpoints
Stage 5 — SPA: Lineage graph (React Flow + dagre + table-level)
Stage 6 — SPA: Column-level lineage + column table nodes
Stage 7 — SPA: Impact analysis page + edge detail panel + polish
```

---

## Stage 1 — API: Asset Catalog, Detail, and Attributes

**What:** Add asset routers to the WMA. Three new endpoints that read from Harvester MongoDB collections.

**Endpoints:**
- `GET /api/assets` — paginated, filterable, searchable asset list with facets
- `GET /api/assets/{jrn}` — single asset with lineage summary
- `GET /api/assets/{jrn}/attributes` — paginated column/field list

**MongoDB indexes to create:**
- `data_distributions`: text index on `name`, compound indexes on `(platform, source.sourceStatus)`, `(source.sourceTypeName)`, `(environment)`, unique on `(jrn)`
- `data_attributes`: compound on `(parent_jrn, ordinal)`
- `lineage_events`: compound on `(source_jrn, is_column_level, confidence)` and `(target_jrn, is_column_level, confidence)`

**Files created:**
```
wma/
  routers/
    assets.py               — FastAPI router for asset endpoints
  services/
    asset_service.py         — query logic, facet aggregation
  models/
    asset_models.py          — Pydantic response models
  tests/
    test_assets_router.py
    test_asset_service.py
```

**Depends on:** WMA project existing with FastAPI + MongoDB client configured.

### Stage 1 — Claude Code Prompt

> Add asset browsing API endpoints to the Workflow Management API (WMA). Reference `docs/asset_lineage_module_architecture.md` sections 3.1, 3.2, 3.3, and 6 for the full API specification, response shapes, and MongoDB indexes.
>
> **Context:** The WMA is a FastAPI application that already has routers under `routers/` and services under `services/`. MongoDB is already connected via a `db` dependency. The Harvester writes to `data_distributions`, `data_attributes`, and `lineage_events` collections — these endpoints are **read-only** against those collections.
>
> Create:
>
> 1. `routers/assets.py` — FastAPI router with three endpoints:
>    - `GET /api/assets` — asset catalog with full-text search (`q` param), platform/type/status/environment filters, pagination (`page`, `page_size`), sorting (`sort`, `order`). Response includes `total`, `page`, `page_size`, `items[]` (each with jrn, name, platform, environment, type_name, schema, catalog, status, owner, last_modified, has_lineage, lineage_edge_count, attribute_count), and `facets` (platform counts, environment counts, type counts). Facets are computed via MongoDB aggregation pipeline `$facet` stage.
>    - `GET /api/assets/{jrn}` — single asset detail. URL-decode the JRN path parameter. Include a `lineage_summary` sub-object with `upstream_count`, `downstream_count`, and `column_lineage_available` (check if any `lineage_events` with `is_column_level: true` exist for this JRN).
>    - `GET /api/assets/{jrn}/attributes` — paginated column/field list for the asset. Query `data_attributes` by `parent_jrn`, sorted by `ordinal`. Each item includes `has_column_lineage` (check if any column-level `lineage_events` reference this column name for the parent JRN).
>
> 2. `services/asset_service.py` — query logic separated from the router:
>    - `search_assets(q, filters, pagination, sort)` — builds MongoDB query with `$text` search when `q` is provided, `$and` filters for platform/type/status/environment, `$skip`/`$limit` for pagination, and a parallel `$facet` aggregation for filter counts.
>    - `get_asset(jrn)` — single document lookup + lineage summary.
>    - `get_attributes(jrn, pagination)` — attribute list with column lineage check.
>
> 3. `models/asset_models.py` — Pydantic response models:
>    - `AssetListResponse`, `AssetItem`, `AssetFacets`, `FacetCount`
>    - `AssetDetailResponse`, `LineageSummary`
>    - `AttributeListResponse`, `AttributeItem`
>
> 4. MongoDB indexes — create a setup function (or migration script) that ensures all indexes from the architecture doc section 6 exist. Call it on app startup.
>
> 5. Tests in `tests/`:
>    - `test_assets_router.py` — 10 tests: search with query, platform filter, pagination, sorting, facet counts, single asset detail, lineage summary counts, attributes list, attributes pagination, 404 on unknown JRN.
>    - `test_asset_service.py` — 8 tests: query builder with text search, compound filter, empty filters return all, facet aggregation, lineage summary with/without edges, attribute column lineage flag.
>
> Use `mongomock` or insert test fixtures into a test MongoDB database. The router tests should use FastAPI `TestClient`. Run pytest green before finishing.

---

## Stage 2 — SPA: Asset Catalog Page

**What:** Add the asset catalog page to the SPA with search, filters, and paginated results.

**Pages:** `/assets`

**Components:**
- `AssetCatalogPage.tsx` — page wrapper
- `AssetSearch.tsx` — search input with 300ms debounce
- `AssetFilters.tsx` — platform, type, status, environment filter dropdowns driven by facet counts
- `AssetTable.tsx` — paginated table with platform badge, name, type, lineage edge count, attribute count
- `PlatformBadge.tsx` — colored badge per platform

**Dependencies:** React, TanStack Query (or SWR), React Router

### Stage 2 — Claude Code Prompt

> Add the Asset Catalog page to the Control Plane SPA. Reference `docs/asset_lineage_module_architecture.md` section 4.2 (Asset Catalog Page wireframe) for the layout.
>
> **Context:** The SPA is a React application using TypeScript, React Router for navigation, and TanStack Query for data fetching. The existing navigation has a "Workflows" section. Add a parallel "Assets" section.
>
> Create under `src/modules/assets/`:
>
> 1. `api/assetApi.ts` — API client functions:
>    - `fetchAssets(params: AssetSearchParams): Promise<AssetListResponse>` — calls `GET /api/assets` with query, filters, pagination
>    - `fetchAsset(jrn: string): Promise<AssetDetailResponse>` — calls `GET /api/assets/{jrn}`
>    - `fetchAttributes(jrn: string, page: number): Promise<AttributeListResponse>` — calls `GET /api/assets/{jrn}/attributes`
>    - Base URL from environment variable `REACT_APP_API_URL`
>
> 2. `types/asset.ts` — TypeScript types matching the API response shapes from Stage 1:
>    - `AssetSearchParams`, `AssetListResponse`, `AssetItem`, `AssetFacets`, `FacetCount`
>    - `AssetDetailResponse`, `LineageSummary`
>    - `AttributeListResponse`, `AttributeItem`
>
> 3. `hooks/useAssets.ts` — TanStack Query hook:
>    - `useAssets(params)` — wraps `fetchAssets`, returns `{ data, isLoading, error }`, refetches on param change
>
> 4. `pages/AssetCatalogPage.tsx` — the main page:
>    - Search bar at top with `AssetSearch` component (300ms debounce on input)
>    - Filter row with `AssetFilters` — four dropdowns (Platform, Type, Status, Environment) each showing options with counts from the `facets` response
>    - Results area with `AssetTable` — columns: Platform (badge), Name (linked to detail page), Type, Schema, Status, Lineage Edges, Attributes
>    - Pagination at bottom (page numbers + prev/next)
>    - Total count shown above results ("4,518 assets")
>    - URL params sync: filters and search query reflected in URL so the page is shareable/bookmarkable
>
> 5. `components/AssetSearch.tsx` — input with magnifying glass icon, 300ms debounce, clears on X button
>
> 6. `components/AssetFilters.tsx` — row of select dropdowns. Each shows "All {type}" as default. Options populated from `facets` response with count in parentheses, e.g. "DATABRICKS (3,200)". Selecting a filter updates URL params and triggers refetch.
>
> 7. `components/AssetTable.tsx` — table with sortable column headers (click to sort). Name column is a `<Link>` to `/assets/:jrn` (URL-encode the JRN). "Lineage Edges" column shows count with a small lineage icon if > 0.
>
> 8. `components/PlatformBadge.tsx` — small colored badge:
>    - DATABRICKS: `#FF3621` background
>    - SNOWFLAKE: `#29B5E8` background
>    - TABLEAU: `#E97627` background
>    - ORACLE: `#F80000` background
>    - ALTERYX: `#0078C0` background
>    - White text, rounded corners, uppercase platform name
>
> 9. Add route to app router: `/assets` → `AssetCatalogPage`
>
> 10. Add "Assets" link to the main navigation sidebar, below "Workflows"
>
> Use Tailwind CSS for styling. Keep it clean and professional — no heavy theming. Empty state: "No assets found" with a hint to check filters. Loading state: skeleton rows in the table.

---

## Stage 3 — SPA: Asset Detail Page

**What:** Add the asset detail page with Overview and Attributes tabs. Lineage tab is a placeholder — built in Stage 5.

**Pages:** `/assets/:jrn`

**Components:**
- `AssetDetailPage.tsx` — page with tabs
- `AssetOverview.tsx` — metadata grid + lineage summary card
- `AttributeTable.tsx` — column list with types and lineage indicator

### Stage 3 — Claude Code Prompt

> Add the Asset Detail page to the Control Plane SPA. Reference `docs/asset_lineage_module_architecture.md` section 4.2 (Asset Detail Page wireframe) for the layout.
>
> **Context:** Stage 2 is complete — the asset catalog page exists and links to `/assets/:jrn`. The `assetApi.ts` and types already exist.
>
> Create under `src/modules/assets/`:
>
> 1. `hooks/useAssetDetail.ts` — TanStack Query hook wrapping `fetchAsset(jrn)`. Decodes the JRN from the URL param.
>
> 2. `hooks/useAttributes.ts` — TanStack Query hook wrapping `fetchAttributes(jrn, page)`. Paginated.
>
> 3. `pages/AssetDetailPage.tsx`:
>    - Breadcrumb: `Assets / {asset_name}`
>    - Header: qualified name, badges for TYPE, PLATFORM, ENVIRONMENT, STATUS
>    - Tab bar: Overview | Attributes ({count}) | Lineage
>    - Overview tab renders `AssetOverview`
>    - Attributes tab renders `AttributeTable`
>    - Lineage tab renders a placeholder: "Lineage visualization coming soon" with a lineage icon (will be replaced in Stage 5)
>    - Tab state in URL hash (`#overview`, `#attributes`, `#lineage`) so direct linking works
>
> 4. `components/AssetOverview.tsx`:
>    - Metadata grid (2 columns):
>      - Left: Owner, Catalog, Schema, SEAL Node, Description
>      - Right: Last Modified, Last Crawled, Created, JRN (copyable)
>    - Lineage Summary card below:
>      - "{N} upstream tables → [THIS TABLE] → {N} downstream consumers"
>      - "Column lineage available ✓" or "Column lineage not available"
>      - Two buttons: "View Full Lineage" (switches to lineage tab), "Impact Analysis" (links to `/assets/:jrn/impact`)
>
> 5. `components/AttributeTable.tsx`:
>    - Columns: #, Name, Data Type, Nullable, Description, Lineage
>    - # = ordinal position
>    - Name = column name (monospace font)
>    - Data Type = raw_data_type (monospace font, muted color)
>    - Nullable = Yes/No badge
>    - Lineage = "⇄ {N} edges" linked to column lineage (Stage 6), or "—" if no lineage
>    - Pagination if > 100 attributes
>    - Sort by ordinal (default), name, or data type
>
> 6. Add route: `/assets/:jrn` → `AssetDetailPage`
>
> Style: Tailwind CSS. Keep consistent with the catalog page. Loading states: skeleton blocks for metadata, skeleton rows for attributes. Error state: "Asset not found" with link back to catalog.

---

## Stage 4 — API: Lineage Graph and Impact Analysis

**What:** Add lineage graph assembly and impact analysis endpoints to the WMA.

**Endpoints:**
- `GET /api/lineage/graph` — BFS traversal, returns React Flow nodes + edges
- `GET /api/lineage/impact` — downstream BFS, returns impacted asset list

### Stage 4 — Claude Code Prompt

> Add lineage graph and impact analysis API endpoints to the WMA. Reference `docs/asset_lineage_module_architecture.md` sections 3.4, 3.5, 5.1, and 5.2 for the full API specification, graph assembly algorithm, and impact analysis service.
>
> **Context:** Stage 1 is complete — the WMA has asset routers and a MongoDB client. The `lineage_events` collection has the indexes from Stage 1.
>
> Create:
>
> 1. `routers/lineage.py` — FastAPI router:
>    - `GET /api/lineage/graph` — parameters: `jrn` (required), `depth` (default 2, max 5), `direction` (upstream|downstream|both, default both), `min_confidence` (default 0.0), `column_level` (default false), `column` (optional, specific column name). Returns `LineageGraphResponse` with `root_jrn`, `depth`, `direction`, `nodes[]`, `edges[]`, `column_edges[]`, `stats`.
>    - `GET /api/lineage/impact` — parameters: `jrn` (required), `depth` (default 5, max 10), `min_confidence` (default 0.5). Returns `ImpactAnalysisResponse` with `root_jrn`, `impacted_assets[]`, `total_impacted`, `max_distance`.
>
> 2. `services/lineage_graph_service.py` — the graph assembly service:
>    - `build_graph(start_jrn, depth, direction, min_confidence, column_level)` — BFS traversal using a deque. For each node, query `lineage_events` by `source_jrn` (downstream) and/or `target_jrn` (upstream), filtered by `is_column_level` and `confidence >= min_confidence`. Collect visited JRNs. Deduplicate edges by edge ID.
>    - `_enrich_nodes(jrns, root_jrn, include_columns)` — batch lookup visited JRNs in `data_distributions`. For each, build a React Flow node with `id`, `type` ("tableNode" or "columnTableNode"), `position: {x:0, y:0}`, and `data` containing label, platform, schema, typeName, environment, status, isRoot. If `include_columns`, also load `data_attributes` for each node and include columns array.
>    - `_format_edges(raw_edges, column_level)` — convert MongoDB edge docs to React Flow edge format. Each edge has `id`, `source`, `target`, `animated` (true if confidence < 0.5), `data` (tier, confidence, queryType, provenance, corroboratedBy, firstSeen, lastConfirmed), `style` (confidence-based color: ≥0.9 green `#16a34a`, ≥0.5 amber `#ca8a04`, <0.5 red dashed `#dc2626`). For column-level edges, include `sourceHandle` and `targetHandle`.
>    - `_count_by_tier(edges)` — count edges per tier for stats.
>
> 3. `services/impact_analysis_service.py` — downstream BFS:
>    - `analyze(start_jrn, depth, min_confidence)` — BFS traversal following only downstream edges (`source_jrn` matches current). Track path for each visited node. Enrich with metadata from `data_distributions`. Sort by distance ascending. Classify as "direct" (distance=1) or "indirect" (distance>1).
>
> 4. `models/lineage_models.py` — Pydantic response models:
>    - `LineageGraphResponse`, `GraphNode`, `GraphNodeData`, `GraphEdge`, `GraphEdgeData`, `GraphEdgeStyle`, `ColumnInfo`, `GraphStats`
>    - `ImpactAnalysisResponse`, `ImpactedAsset`
>
> 5. Tests:
>    - `test_lineage_router.py` — 8 tests: graph with depth=1, depth=2, upstream only, downstream only, min_confidence filter, column_level=true, impact analysis, empty graph for isolated node.
>    - `test_lineage_graph_service.py` — 10 tests: BFS visits correct nodes, deduplicates edges, enriches from distributions, handles missing distributions gracefully, respects depth limit, confidence filtering, column edges include handles, edge styling by confidence, stats counts, cross-platform edges.
>    - `test_impact_analysis_service.py` — 6 tests: direct impacts distance=1, indirect chain, respects depth limit, min_confidence filtering, path tracking, empty downstream.
>
> Insert test fixtures: create 5 distributions and 8 lineage_events (mix of tiers and column-level) in test setup. Run pytest green.

---

## Stage 5 — SPA: Lineage Graph (Table-Level)

**What:** Replace the lineage tab placeholder with the React Flow graph visualization for table-level lineage.

**Dependencies:** `reactflow`, `@dagrejs/dagre`

### Stage 5 — Claude Code Prompt

> Build the lineage visualization in the Control Plane SPA using React Flow. Reference `docs/asset_lineage_module_architecture.md` sections 4.2 (Lineage Tab wireframe) and 7.1 (React components).
>
> **Context:** Stages 2–3 are complete — the asset detail page exists with a placeholder lineage tab. Stage 4 is complete — the API returns graph data at `GET /api/lineage/graph`. Install `reactflow` and `@dagrejs/dagre`.
>
> Create under `src/modules/assets/`:
>
> 1. `types/lineage.ts` — TypeScript types matching the API lineage response:
>    - `LineageGraphParams`, `LineageGraphResponse`, `GraphNode`, `GraphNodeData`, `GraphEdge`, `GraphEdgeData`, `ColumnInfo`, `GraphStats`
>
> 2. `api/assetApi.ts` — add:
>    - `fetchLineageGraph(params: LineageGraphParams): Promise<LineageGraphResponse>`
>
> 3. `hooks/useLineageGraph.ts` — TanStack Query hook wrapping `fetchLineageGraph`. Refetches when params change.
>
> 4. `utils/layout.ts` — dagre layout function:
>    - `applyDagreLayout(nodes: Node[], edges: Edge[]): Node[]`
>    - Direction: left-to-right (`rankdir: 'LR'`), `ranksep: 120`, `nodesep: 40`
>    - Node dimensions: 220px wide, 60px tall for table nodes
>    - Returns nodes with computed `position` from dagre
>
> 5. `components/TableNode.tsx` — custom React Flow node for table-level view:
>    - Header: PlatformBadge + table name (bold)
>    - Sub-line: qualified name (muted, smaller text)
>    - Root node: highlighted border (blue)
>    - Status indicator: green dot for ACTIVE, red for DELETED
>    - Left Handle (target) and Right Handle (source)
>    - Click navigates to that asset's detail page
>
> 6. `components/LineageControls.tsx` — control panel in a floating card:
>    - Direction: select dropdown (Both, Upstream Only, Downstream Only)
>    - Depth: select dropdown (1, 2, 3, 4, 5)
>    - Min Confidence: select dropdown (All, ≥0.5, ≥0.9)
>    - View toggle: Table Level / Column Level (radio buttons, column level disabled in this stage)
>    - Fit View button, Reset button
>    - Stats display: "{N} tables, {N} edges" from response stats
>
> 7. `components/LineageLegend.tsx` — small floating legend:
>    - Solid green line (2px): Tier 1 — System tables (confidence ≥ 0.9)
>    - Solid amber line (1.5px): Tier 2/3 — Moderate confidence (≥ 0.5)
>    - Dashed red line (1px): Low confidence (< 0.5)
>    - Star icon: Root asset
>
> 8. `components/LineageGraph.tsx` — the main React Flow wrapper:
>    - Loads data via `useLineageGraph` hook
>    - Applies dagre layout when data changes
>    - Registers custom node types: `{ tableNode: TableNode }`
>    - Includes Background (dots), Controls (zoom), MiniMap
>    - `fitView` on initial load
>    - Click on an edge → sets `selectedEdge` state (for Stage 7 EdgeDetailPanel)
>    - Loading state: centered spinner
>    - Empty state: "No lineage found for this asset"
>
> 9. Replace the placeholder in `AssetDetailPage.tsx` lineage tab with `<LineageGraph jrn={asset.jrn} />`
>
> Style: Clean, professional. Node cards with subtle shadow and rounded corners. Use Tailwind. The graph area should fill the available height (min 500px). Controls and legend are semi-transparent floating panels.

---

## Stage 6 — SPA: Column-Level Lineage

**What:** Add column-level lineage visualization with column handles on table nodes.

### Stage 6 — Claude Code Prompt

> Add column-level lineage visualization to the SPA. Reference `docs/asset_lineage_module_architecture.md` section 7.1 (ColumnTableNode).
>
> **Context:** Stage 5 is complete — table-level lineage works with React Flow. The API already supports `column_level=true` which returns nodes with `columns[]` and `column_edges[]` with `sourceHandle`/`targetHandle`.
>
> Modify and create under `src/modules/assets/`:
>
> 1. `components/ColumnTableNode.tsx` — new custom React Flow node:
>    - Header: PlatformBadge + table name (same as TableNode)
>    - Below header: list of columns, each as a row with:
>      - Left Handle (target) with `id={column.name}` and `Position.Left`
>      - Column name (monospace)
>      - Column type (muted, smaller)
>      - Right Handle (source) with `id={column.name}` and `Position.Right`
>    - Handles are small circles, aligned with each column row
>    - Root node highlighted border
>    - Max 20 columns shown; if more, show "+N more" with expand button
>
> 2. Update `utils/layout.ts`:
>    - `applyDagreLayout` must calculate node height dynamically: base height (60px) + column count × 24px per row
>    - Pass actual height to `g.setNode()` so dagre spaces column nodes correctly
>
> 3. Update `components/LineageGraph.tsx`:
>    - Register `columnTableNode` in `nodeTypes`
>    - When `params.columnLevel` is true:
>      - Use `data.column_edges` (in addition to or instead of `data.edges`) for the edge array
>      - Column edges have `sourceHandle` and `targetHandle` — React Flow draws them between specific column handles
>    - Combine table-level edges and column-level edges: table edges connect node borders, column edges connect specific handles
>
> 4. Update `components/LineageControls.tsx`:
>    - Enable the "Column Level" radio button (was disabled in Stage 5)
>    - When switched to column level, add a column name input: "Column: [{name}]" (optional — filters to lineage for one specific column using the `column` API param)
>    - Switching view triggers a new API call with `column_level=true`
>
> 5. Update `components/AttributeTable.tsx`:
>    - The "⇄ N edges" link in the Lineage column now navigates to the lineage tab with `column_level=true&column={name}` pre-selected
>
> Handle edge routing: column edges from a source column handle to a target column handle. React Flow's default edge routing works but for dense column maps, use `smoothstep` edge type for cleaner paths:
> ```tsx
> { ...edge, type: 'smoothstep' }
> ```
>
> Test: verify column nodes render with correct handle count, column edges connect to correct handles, switching between table/column view re-layouts correctly.

---

## Stage 7 — SPA: Impact Analysis + Edge Detail + Polish

**What:** Add the edge detail panel, impact analysis page, and overall polish.

### Stage 7 — Claude Code Prompt

> Complete the asset explorer module with impact analysis and polish. Reference `docs/asset_lineage_module_architecture.md` sections 3.5 and 4.2.
>
> **Context:** Stages 2–6 are complete. The lineage graph works at both table and column level. The API impact analysis endpoint exists from Stage 4.
>
> Create and modify under `src/modules/assets/`:
>
> 1. `api/assetApi.ts` — add:
>    - `fetchImpactAnalysis(jrn: string, depth: number, minConfidence: number): Promise<ImpactAnalysisResponse>`
>
> 2. `types/asset.ts` — add: `ImpactAnalysisResponse`, `ImpactedAsset`
>
> 3. `hooks/useImpactAnalysis.ts` — TanStack Query hook wrapping `fetchImpactAnalysis`.
>
> 4. `components/EdgeDetailPanel.tsx` — sliding panel that appears when an edge is clicked in the lineage graph:
>    - Source → Target (both linked to their asset detail pages)
>    - Tier badge: "Tier 1 — System Tables" / "Tier 2 — OpenLineage" / "Tier 3 — Query Parse"
>    - Confidence: numeric value + visual bar
>    - Query type: INSERT / MERGE / CTAS / etc.
>    - Provenance: list of sources that found this edge
>    - Corroborated by: list of additional sources (if any)
>    - First seen: date
>    - Last confirmed: date
>    - "View Column Lineage" button (switches to column-level view for this edge)
>    - Close (X) button
>    - Slides in from the right, 320px wide, semi-transparent background overlay
>
> 5. `components/ConfidenceBadge.tsx` — small colored badge:
>    - ≥ 0.9: green, "High"
>    - ≥ 0.5: amber, "Medium"
>    - < 0.5: red, "Low"
>    - Shows numeric value on hover
>
> 6. `pages/ImpactAnalysisPage.tsx` — accessible from `/assets/:jrn/impact`:
>    - Header: "Impact Analysis for {asset_name}"
>    - Controls: Depth slider (1–10), Min Confidence dropdown
>    - Summary: "{N} assets impacted, max {N} hops downstream"
>    - Results table with `ImpactTable` component:
>      - Columns: Distance, Name (linked), Platform, Impact Type (direct/indirect badge), Path
>      - Path shown as breadcrumb: "trades → positions → risk_dashboard"
>      - Sorted by distance ascending
>    - Mini lineage graph showing only the downstream paths (direction=downstream)
>
> 7. `components/ImpactTable.tsx` — the impact results table:
>    - Distance column: numeric badge (1, 2, 3...)
>    - Impact type: green "Direct" badge for distance=1, amber "Indirect" for distance>1
>    - Path: clickable breadcrumb, each segment links to the asset detail page
>    - Empty state: "No downstream assets found"
>
> 8. Wire `EdgeDetailPanel` into `LineageGraph.tsx`:
>    - `onEdgeClick` callback sets the selected edge
>    - Panel renders over the graph
>    - Clicking background or X closes the panel
>
> 9. Update `AssetOverview.tsx`:
>    - "View Full Lineage" button navigates to `#lineage` tab
>    - "Impact Analysis" button navigates to `/assets/:jrn/impact`
>
> 10. Add route: `/assets/:jrn/impact` → `ImpactAnalysisPage`
>
> 11. Polish:
>    - Loading states: skeleton loaders for all data-dependent sections
>    - Error states: "Failed to load" with retry button
>    - Empty states: meaningful messages for each empty scenario
>    - Keyboard: Escape closes the edge detail panel
>    - Responsive: graph controls stack vertically on narrow viewports
>    - Transitions: panel slides in, graph fades in
>
> Test: verify edge click opens panel with correct data, impact analysis shows correct distance, path breadcrumbs link correctly, empty states render.

---

## Summary — Implementation Timeline

| Stage | What | Effort | Depends on |
|---|---|---|---|
| 1 | API: Asset catalog + detail + attributes | M (3–4 days) | WMA exists |
| 2 | SPA: Asset catalog page | M (3–4 days) | Stage 1 |
| 3 | SPA: Asset detail page | M (3–4 days) | Stage 1, 2 |
| 4 | API: Lineage graph + impact analysis | L (5–7 days) | Stage 1 |
| 5 | SPA: Lineage graph (table-level) | L (5–7 days) | Stage 3, 4 |
| 6 | SPA: Column-level lineage | M (3–4 days) | Stage 5 |
| 7 | SPA: Impact analysis + edge detail + polish | M (4–5 days) | Stage 5, 6 |
| **Total** | | **~26–35 days** | |

**In sprints:** approximately 3–4 sprints (2-week sprints, single developer).

**Recommended execution order:**

```
Week 1:  Stage 1 (API) + Stage 2 (Catalog page)
Week 2:  Stage 3 (Detail page) + Stage 4 (Lineage API)
Week 3:  Stage 5 (React Flow table-level graph)
Week 4:  Stage 6 (Column-level) + Stage 7 (Impact + polish)
```

Each week produces a deployable increment:
- End of Week 1: users can search and browse assets
- End of Week 2: users can view asset detail with columns
- End of Week 3: users can see table-level lineage graphs
- End of Week 4: full column-level lineage + impact analysis
