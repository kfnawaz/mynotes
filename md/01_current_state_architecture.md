# Current-State Architecture

## Purpose

This page documents the current architecture direction for Data Compass AI Modernization.

## Current Logical Flow

```
Source Metadata
→ Data Compass Ingestion Service
→ Canonical / Normalized Metadata Model
→ MongoDB Document Store
→ MongoDB Vector Indexes
→ Neo4j Graph Projection
→ Data Compass Search Service
→ Context Plane / Agents
→ Data Compass UI / Answer Plane
```

## Source Systems

- Snowflake
- Databricks
- Tableau
- Alteryx
- Data Compass API
- Harvester / Crawler

## Core Storage Components

### MongoDB

MongoDB stores the authoritative Data Compass metadata model.

Responsibilities:
- Store catalog entities
- Store canonical metadata records
- Store AI-ready chunks, if using MongoDB as the vector document store
- Store source **guid**, **jrn**, lifecycle, ownership, classification, and entity metadata

### MongoDB Atlas Vector Search

Responsibilities:
- Store embeddings
- Support semantic search
- Support metadata-filtered vector search
- Support AI chunk retrieval

### Neo4j

Neo4j is a derived graph projection.

Responsibilities:
- Store lineage relationships
- Store asset relationships
- Support upstream/downstream traversal
- Support impact analysis
- Support GraphRAG graph expansion

## Data Compass Search Service

Responsibilities:
- Exact metadata search
- Semantic vector search
- Graph traversal search
- Hybrid search
- Source hydration
- Citation/source output

## Context Plane

The Context Plane supports:
- Acronyms
- Historical Q&A
- Human feedback
- User bookmarks
- Self-evaluation
- Supervisor Agent
- Structured Discovery Agent
- Metadata Enrichment Agent

## Known Gaps

| Gap | Impact | Owner | Status |
| --- | --- | --- | --- |
| MVP entity scope needs formal approval | Blocks Phase 1 build | TBD | Open |
| JRN construction and identity rules need to be finalized | Blocks chunking and graph projection | TBD | Open |
| Search security filtering model needs to be defined | Blocks production readiness | TBD | Open |
| Graph projection schema needs to be finalized | Blocks Neo4j build | TBD | Open |
| Chunk schema and embedding strategy need to be approved | Blocks vector indexing | TBD | Open |
