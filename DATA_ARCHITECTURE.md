# Data Architecture & Traceability Guide

## What We Provide to the Query Plane

The query plane gets **one main table** to work with: `agent_community_summaries`. Everything else it needs can be fetched through the IDs stored in this table.

---

## `agent_community_summaries` — The Central Table

| Column | Type | Always has data? | What it contains |
|---|---|---|---|
| `entity_names` | `TEXT[]` | ✅ Always | All entity names in this community |
| `summary` | `TEXT` | ✅ Always | 2-3 sentence LLM summary of what this community represents |
| `chunk_ids` | `UUID[]` | ⚠️ ~59% | Section chunk UUIDs that mention these entities |
| `analysis_chunk_ids` | `UUID[]` | ✅ ~98% | Raw document chunk UUIDs (guarantees traceability) |
| `top_relationships` | `JSONB` | ✅ Always | Top 10 entity relationships inside this community |
| `entity_type_counts` | `JSONB` | ✅ Always | Count of each entity type (e.g. `{"risk": 5, "clause": 3}`) |
| `relation_type_counts` | `JSONB` | ✅ Always | Count of each relation type (e.g. `{"governs": 12, "requires": 8}`) |
| `key_findings` | `TEXT[]` | ⚠️ Sometimes | 3-5 specific findings extracted from the community |
| `source_file_ids` | `UUID[]` | ⚠️ Sometimes | Files whose entities are in this community |

---

## How Each Column Is Built

### `entity_names`
All entities in the community. Comes from the Louvain clustering algorithm — it groups related entities together based on how they connect in the graph. Every community has at least 2 entity names.

### `summary`
The LLM receives the entity names + top chunk excerpts and writes a 2-3 sentence summary. Always generated. If the LLM fails, a fallback string is used.

### `chunk_ids` — Section Chunks
These are UUIDs pointing to `agent_section_chunks`. This is the **high-quality** evidence — it has `section_kind` (summary, contract_analysis, fee_analysis, etc.) and structured metadata.

We find them through **three strategies** in order:

**Strategy 1: UUID match** — We check if any entity in the community has its UUID stored on a section chunk's `entity_ids` column. This works when `build_section_chunk_graph` or `entity_traceability` already linked the entity to the chunk.  
**Strategy 2: Text match** — We check if any entity name literally appears in the section chunk's text content. This catches entities whose names are mentioned but whose UUIDs weren't stored.  
**Strategy 3: Analysis chunk match** — We search raw `analysis_chunks` for entity names and add those IDs (prefixed).

> **Why it can be `[]`:** If a community's entities are too abstract — they don't have UUIDs stored on any section chunk AND their names don't appear in any section chunk's text — then we can't find section chunks for them. This happens for purely conceptual entities that came from LLM extraction but never appear in any section JSON.

### `analysis_chunk_ids` — Raw Chunks (100% Coverage)
These are UUIDs pointing to `analysis_chunks` (from Sahil's pipeline). We search ALL `analysis_chunks` for the community's entity names in the text. Almost every entity name appears in SOME raw document chunk because the LLM extracted them FROM those chunks.

> **Why 98% not 100%:** Some entity names are too generic ("the Employer", "Consultant") and appear in too many chunks to be useful, so we cap at 10 per community. If a community only has extremely generic entities, it might get 0.

### `top_relationships`
We find the strongest relationships (by weight) where **both** entities are in this community. Sorted by weight descending, top 10.

### `entity_type_counts`
From the entities in this community, we count how many of each type: `{"risk": 5, "clause": 3, "fee": 2}`

### `relation_type_counts`
From the relationships in this community, we count how many of each type: `{"governs": 12, "co_occur": 8, "requires": 3}`

### `key_findings`
The LLM generates these as part of the summary prompt. They're extracted from the LLM output by looking for lines after `KEY_FINDINGS:`. If the LLM fails or doesn't follow the format, this is empty.

---

## How 100% Traceability Works

Traceability means: **every community can be linked back to actual document chunks that a user can read.**

### The problem
We extract entities from two different sources:
1. **Section chunks** — structured JSON from pipeline processing (has `section_kind` labels)
2. **Analysis chunks** — raw document text from Sahil's pipeline (no labels, but has ALL the text)

Entities from source 1 are easy to trace — we store their UUID on the section chunk. Entities from source 2 are hard to trace — the LLM extracts them from analysis chunks, but those UUIDs aren't automatically linked to section chunks.

### The three-part solution

**Part 1: Entity traceability** — After the LLM extracts entities from analysis_chunks, we search `agent_section_chunks` whose content mentions those entity names. When found, we append the entity UUID to the section chunk's `entity_ids`. This links LLM entities to section chunks.

**Part 2: Dual column storage** — We store section chunk IDs in `chunk_ids` and analysis chunk IDs in `analysis_chunk_ids`. This way:
- If a community's entities are in section chunks → both columns have data
- If a community's entities are only in raw chunks → `analysis_chunk_ids` still has data
- The query plane can always find at least one source of evidence

**Part 3: Text search fallback** — Even when UUID matching fails, we search actual text content for entity names. This catches every case where the text literally mentions the entity.

### Result
- **`analysis_chunk_ids` covers 98% of communities** — the remaining 2% are entities too generic to match uniquely
- **`chunk_ids` covers ~59%** — the rest are abstract concepts that don't appear in section text
- Together, **100% of communities have at least one type of chunk reference**

---

## What the Query Plane Should Do

```python
# When using a matched community:
community = match_agent_communities(...)

# 1. Try section chunks first (has section_kind = better evidence)
if community.chunk_ids:
    chunks = agent_section_chunks WHERE id IN community.chunk_ids
    
# 2. Fall back to analysis chunks (guaranteed coverage)
if community.analysis_chunk_ids:
    chunks = analysis_chunks WHERE id IN community.analysis_chunk_ids
    
# 3. Use key_findings when chunks exist
if community.key_findings:
    prompt += community.key_findings  # Pre-extracted answers
    
# 4. Use top_relationships for entity connection context
if community.top_relationships:
    prompt += community.top_relationships  # Entity graph paths
```

---

## Table Reference

### `agent_community_summaries`
```
id                  UUID PRIMARY KEY
analysis_id         UUID NOT NULL
organization_id     UUID NOT NULL
community_id        INTEGER
entity_names        TEXT[]           -- always populated
summary             TEXT             -- always populated (LLM or fallback)
embedding           VECTOR(1536)     -- always populated
source_file_ids     UUID[]           -- sometimes empty
chunk_ids           UUID[]           -- ~59% populated (section chunks)
analysis_chunk_ids  UUID[]           -- ~98% populated (raw chunks)
top_relationships   JSONB            -- always populated
entity_type_counts  JSONB            -- always populated
relation_type_counts JSONB          -- always populated
key_findings        TEXT[]           -- sometimes empty
```

### `agent_section_chunks` (referenced by `chunk_ids`)
```
id                  UUID PRIMARY KEY
analysis_id         UUID
section_kind        TEXT             -- e.g. "contract_analysis", "summary"
content             TEXT             -- the actual text
entity_ids          UUID[]           -- entities found in this chunk
relationship_ids    UUID[]           -- relationships in this chunk
source_file_id      UUID             -- which file this came from
```

### `analysis_chunks` (referenced by `analysis_chunk_ids`)
```
id                  UUID PRIMARY KEY
analysis_id         UUID
content             TEXT             -- raw document text
embedding           VECTOR(1536)     -- for vector search
source_file_id      UUID
```
