# Internship Documentation

> High-level overview of work completed during this internship period.
> No proprietary code, client names, or confidential business logic is included.
> All descriptions are of technologies, architectures, and problem domains only.

---

## 📑 Version Toggle

> Click to switch between documentation periods:

<details open>
<summary><strong>📄 README v1 — June 15 to July 31, 2026</strong> (currently shown — the full document below)</summary>

</details>

<details>
<summary><strong>📄 README v2 — August 2026 onwards</strong> (click to expand)</summary>

### Internship Documentation — August 2026 onwards

> This section documents work from **August 1, 2026** onwards.
> Content will be added here as the work progresses. See [`readme2.md`](readme2.md) for the standalone file.

</details>

---

## Table of Contents

1. [Overview](#overview)
2. [Timeline](#timeline)
   - [Phase 1: Ingestion Pipeline & Knowledge Graph (Jun 15 – Jul 9)](#phase-1-ingestion-pipeline--knowledge-graph)
   - [Phase 2: Documentation & Handoff (Jul 7 – Jul 14)](#phase-2-documentation--handoff)
   - [Phase 3: RAG Query Agent (Jul 16 – Jul 23)](#phase-3-rag-query-agent)
   - [Phase 4: Cloud Deployment & Production Hardening (Jul 24 – Jul 27)](#phase-4-cloud-deployment--production-hardening)
   - [Phase 5: RFP Data Pipeline & Production Hardening (Jul 28 – Jul 29)](#phase-5-rfp-data-pipeline--production-hardening)
   - [Phase 6: Initial Pipeline Planning & n8n Bottleneck Analysis (Jul 30)](#phase-6-initial-pipeline-planning--n8n-bottleneck-analysis)
   - [Phase 7: Large-Batch Stress Test & Scalability Fixes (Jul 31)](#phase-7-large-batch-stress-test--scalability-fixes)
3. [Technologies Used](#technologies-used)
4. [Architecture Patterns](#architecture-patterns)
5. [Key Projects](#key-projects)
6. [Skills Demonstrated](#skills-demonstrated)

---

## Overview

Over approximately 6 weeks, work spanned three major areas:

1. **Document Ingestion Pipeline** — A production-grade, domain-agnostic pipeline that ingests documents, processes them through multiple stages (parse, chunk, LLM-tag, embed, knowledge graph extract), and stores results in a vector database for retrieval. Deployed on AWS with full Terraform infrastructure-as-code.

2. **Knowledge Graph Visualization** — An interactive, web-based graph explorer for navigating entity-relationship graphs extracted from documents. Built with D3.js and a FastAPI backend.

3. **RAG Query Agent** — A cross-project retrieval-augmented generation (RAG) system that answers questions across multiple document sets using hybrid search (vector + keyword + knowledge graph). Includes deterministic evidence arbitration, LLM-based query classification, and structured answer generation.

All deliverables involved: Python/backend engineering, cloud infrastructure (AWS), database design (PostgreSQL + pgvector), LLM integration (OpenRouter), frontend visualization, and comprehensive documentation.

---

## Timeline

### Phase 1: Ingestion Pipeline & Knowledge Graph
*June 15 – July 9, 2026*

**GraphRAG Integration (Jun 15–19)**
- Integrated GraphRAG as an optional retrieval layer alongside hybrid search
- Implemented 3-hop graph traversal using NetworkX for knowledge-graph-enhanced retrieval
- Built evidence arbitration between hybrid search results and graph-based retrieval
- Added LLM-based guard checks to prevent off-topic graph synthesis
- Integrated vector database for storage with pgvector extension

**LLM Auto-Tagging & Batch Processing (Jun 22–29)**
- Implemented batch LLM tagging: 10 document chunks per API call for efficiency
- Added metadata scoring with evidence coverage tracking
- Built entity grounding checker for cross-chunk entity consistency
- Implemented DBSCAN-based entity name consolidation (merges similar entity names)
- Added Louvain community detection on the entity relationship graph
- Created GraphRAG arbitration modes (strict, balanced, relaxed) with configurable thresholds

**Entity Graph Visualization (Jul 5–7)**
- Built FastAPI backend with 11 API endpoints for entity/relationship/chunk queries
- Developed D3.js force-directed graph visualization on Canvas
- **Iteration 1 — Tree Exploration**: scroll-to-grow interface where new entities load at viewport edges as the user pans, with cross-batch edges connecting new nodes to the existing tree. No data cap (Infinity rendering), strong collision forces for clean layout.
- **Iteration 2 — Complete Redesign**: Entity-centric Knowledge Graph Explorer with:
  - Landing screen displaying stats and popular entities
  - Search-driven entity loading (no auto-load)
  - Stable layout during expansion (existing nodes freeze, only new nodes animate)
  - Edge evidence panel showing source chunks
  - Path finder using bidirectional BFS
  - Global search (Cmd+K) across entities, documents, and chunk text
  - Node sizing by entity frequency, smart label rendering
  - Relationship filter with autocomplete suggestions
- Frontend: single HTML file (no build tooling), pure D3.js on Canvas
- Backend: FastAPI serving from PostgreSQL

**Infrastructure & AWS Deployment (Jul 1–9)**
- Built full Terraform IaC for the ingestion pipeline covering 35+ AWS resources
- Infrastructure includes: ECS Fargate (compute), S3 (6 buckets), SQS (job queue + DLQ), Lambda (S3 event trigger), RDS PostgreSQL with pgvector, DynamoDB (job tracking), OpenSearch (text search), Neptune (RDF/SPARQL), Bedrock (embeddings), IAM roles with least-privilege policies
- Docker containerization with multi-architecture build support (ARM64 native + x86_64 via buildx)
- SQS-based continuous polling orchestration with 10s long-poll, auto post-processing
- Dual LLM failover system with automatic fallback on rate limit or exhaustion
- Token bucket rate limiter (8 RPS) for LLM API calls
- Parallel pipeline workers (5x speedup)
- Comprehensive pipeline fixes addressing Docker arch mismatch, S3→Lambda notifications, API key management, ECS deployment racing, CodeBuild CI/CD integration
- Verified end-to-end on both Mac ARM64 (~17s per document) and x86_64 (~37s per document)

**Pipeline Stages**
1. **Parse** — Multi-format: PDF, DOCX, XLSX, PPTX, images (OCR), HTML, plain text
2. **Chunk** — 500-word sliding window with 80-word overlap
3. **LLM Tag** — Batch 10 chunks/call: extracts topics, entities, chunk_type flags
4. **Embed** — Embedding model on Bedrock (1024-dim vectors) with hash-based fallback
5. **Graph Extraction** — LLM extracts entities and relationships → DBSCAN consolidation → Louvain community detection

**End-to-End Documentation (Jul 9)**
- Produced a comprehensive 12-section document covering the entire pipeline from prerequisites to troubleshooting
- Includes 7 resolved infrastructure issues with symptom→cause→fix format

---

### Phase 2: Documentation & Handoff
*July 7 – 14, 2026*

Created a dedicated documentation repository containing:

- **Pipeline Architecture Overview** — Complete system architecture, data flows, and deployment topology
- **API Endpoint Reference** — Full API specification for query plane, ingestion pipeline, and graph visualization
- **Local Development Setup** — Step-by-step guide for Docker Compose local environment
- **IAM Roles Guide** — Least-privilege IAM policy documentation
- **Security Checklist** — Comprehensive AWS security audit
- **Environment Templates** — Configuration templates with all variables
- **Terraform Variables Guide** — Complete variables reference
- **MCP Standalone Architecture** — Model Context Protocol server design
- **MCP Query Plane Integration** — Upgraded MCP server with full query pipeline
- **Graph Visualization Redesign** — Frontend architecture documentation

---

### Phase 3: RAG Query Agent & Production Optimization
*July 16 – 23, 2026*

Built a cross-project RAG query agent capable of answering questions across multiple document collections simultaneously, then production-optimized with 100% traceability and batched pipeline operations.

**Query Pipeline Architecture**
- FastAPI server with PostgreSQL + pgvector backend
- 6 parallel search paths: Vector (pgvector), Keyword (FTS), Section, Community (GraphRAG), Entity Graph, Metadata
- 5 query types classified by LLM: lookup, semantic, relational, multi-hop, aggregation
- BM25 re-ranking with relevance boost

**Query Classification & Routing**
- LLM-based query analyser classifies queries into 5 types, extracts keywords/entities, decomposes complex queries
- Project scope resolution via LLM (determining which document collections to search)
- Rule-based fallback when LLM classification fails

**Evidence Arbitration**
- Deterministic arbitration layer across all search paths
- Three modes: strict (≥45% term coverage), balanced (≥30%), relaxed (≥18%)
- Pre-LLM guard checks: project name validation, surface term matching, section kind verification

**Answer Generation**
- Structured answer format with evidence citations
- Project-diverse evidence re-ordering prevents truncation
- Metadata chunks prioritized over narrative chunks
- Evidence-only prompt prevents hallucination

**Systemic Bug Fixes (Jul 22)**
Identified and fixed 7 root-cause issues across the query pipeline:
1. Project diversity enforcement never triggered (comparison operator off-by-one)
2. Context character cap dropped underrepresented projects (re-ordered evidence)
3. Evidence headers showed UUIDs instead of project names (added labels)
4. Answer LLM prompt didn't prefer structured metadata (added priority rules)
5. LLM-generated database filters silently dropped rows (removed unnecessary filters)
6. Missing fallback in lookup route (removed filter entirely)
7. Same issue in semantic route (removed filter entirely)

Verified across 10+ queries of all 5 types with consistent correct results.

**Pipeline Speed & Reliability Optimizations (Jul 23)**
- Switched LLM model from Gemini 2.5 Flash to GPT-5.4-mini for 4x faster inference (0.56s per call)
- Increased LLM concurrency from 5→20 and summary concurrency from 20→50
- Fixed N+1 query bottlenecks in entity dedup and cleanup operations (batched PATCH/DELETE with `IN()` queries):
  - Entity dedup: reduced from 3N API calls to 3 per canonical group
  - Noise removal: reduced from 2N to 2 total calls
  - Relationship ID fetching: reduced from N to 1 per 25 pairs via OR queries
- Fixed null byte crash in Supabase inserts (deep-clean using JSON roundtrip + strip)
- Reduced pipeline time from ~15min to ~2.5min for 5 projects with all fixes combined (LLM phase: 30s vs ~5min)

**Entity Traceability & 100% Community Coverage (Jul 23)**
- Built entity traceability system: after LLM extraction, entity UUIDs are stored back onto section chunks whose content mentions them
- Added `analysis_chunk_ids` column (UUID[]) to community summaries as a guaranteed fallback
- Three-tier chunk matching: UUID match → section text search → analysis chunk text search
- Result: 100% of communities (191/191) have at least one chunk reference; 98% have analysis_chunk_ids

**Community Summary Enrichment (Jul 23)**
- Added 6 new columns to `agent_community_summaries` for richer query plane context:
  - `analysis_chunk_ids UUID[]` — raw chunk references for guaranteed retrieval
  - `top_relationships JSONB` — top 10 entity relationships within community
  - `entity_type_counts JSONB` — distribution of entity types
  - `relation_type_counts JSONB` — distribution of relation types
  - `key_findings TEXT[]` — 3-5 LLM-extracted key findings per community
- Relation normalizer CLUSTER_THRESHOLD reduced from 5→2 to preserve diverse types (41+ types, top type at <26%)

**Lite RAG Agent (Jul 23)**
- Built a lightweight single-project RAG agent with FastAPI router
- 4 search paths: Knowledge Graph, Vector Search, Community Summaries, Metadata Lookup
- BGE reranker (baai/bge-reranker-v2-m3) for relevance scoring across all evidence
- Community routing: matched communities fetch actual section + analysis chunks as evidence
- GraphRAG content search fallback: finds entity names in section chunk text when chunk_id doesn't match

**Cross-Project Agent Updates (Jul 23)**
- Updated community_vector_search to fetch and return analysis_chunk_ids alongside chunk_ids
- Both search paths (vector + relational) now fetch routed chunks from both `agent_section_chunks` and `analysis_chunks`
- All community metadata fields pass through to LLM prompt (top_relationships, key_findings, etc.)

**Schema & Documentation (Jul 23)**
- Migrations for all new columns added to `db/initial_schema.sql`
- Updated `match_agent_communities` RPC to return all new fields
- Created `DATA_ARCHITECTURE.md` documenting the full UUID flow, traceability system, and column-level explanations
- 26 files changed, 2,500+ lines across the codebase

---

### Phase 4: Cloud Deployment & Production Hardening
*July 24 – 27, 2026*

Deployed both the ingestion pipeline and the advanced query plane to AWS ECS Fargate with full automation, idempotency, and fault tolerance.

**Architecture (3-Layer)**

```
Sahil's Frontend → Query Plane (ECS Service + ALB) → Supabase
                                                        ↑
                                              Ingestion Plane (ECS Tasks)
                                              triggered by webhook when analyses.status=complete
```

**Ingestion Plane (Fully Automated)**

1. **Trigger**: Supabase Database Webhook on `analyses` UPDATE where status changes to `'complete'`
2. **Flow**: Webhook → API Gateway → SQS Queue → Lambda Dispatcher → ECS Fargate Task
3. **Lambda Dispatcher**: Checks `agent_ingestion_log` for idempotency, writes a processing lock, then calls `ecs.run_task()`
4. **ECS Task**: 4 vCPU / 8 GB RAM container runs the full 9-stage pipeline (Source B → entity extraction → cleanup → LLM relations → normalization → chunk graph → Louvain communities → community summaries → mark processed)
5. **All results** written to Supabase `agent_*` tables

**Infrastructure Components**
- ECR: Two repositories (`halo-ingestion` for pipeline, `halo-query-plane` for API server)
- ECS Cluster: Shared Fargate cluster
- SQS Queue: Ingestion trigger queue + DLQ
- Lambda Dispatcher: Thin Python function (checks log, writes lock, launches ECS)
- API Gateway: Receives Supabase webhooks and pushes to SQS
- IAM Roles: Least-privilege for each component
- CloudWatch: Centralized logging for all ECS tasks and Lambda functions

**Query Plane (Always-On Service)**

- Advanced query plane module deployed as an always-running FastAPI server on ECS Fargate
- ALB: Application Load Balancer provides stable endpoint
- Endpoint: `POST /query-plane/query`
- Request: `{"query": "...", "org_id": "...", "analysis_id": "..."}`
- Response: `{"answer": "...", "status": "answered|partial|insufficient", "confidence": "high|medium|low", "citations": [...], "projects": [...], "metadata": {...}}`
- Reads from the same Supabase `agent_*` tables that ingestion writes to

**Problems Identified & Fixed (Jul 27)**

| Problem | Root Cause | Fix |
|---|---|---|
| Metadata only 1 row | Hardcoded taggers only work with one specific JSON format. Different pipelines produce different structures. | Changed tagger.py to always run the LLM tagger — works with any JSON format. |
| Duplicate log entries | Lambda wrote a "processing" lock, entrypoint tried to update it but pipeline's `mark_processed` did an INSERT instead. | Entrypoint now writes a single "processing" row at startup and updates to "complete" or "failed" at end. |
| All tasks processed the same analysis | Pipeline polls for ANY unprocessed analysis, not the one passed via env var. | Added `specific_analysis_id` parameter to the orchestrator function. |
| Stuck tasks with no logs | Container crashed before Python logging initialized. No record of failure. | Entrypoint writes "processing" status immediately. Retry Lambda detects stale entries. |
| IAM policy blocked Lambda | Lambda's ECS permission was pinned to an old task definition revision. | Updated policy to use wildcard for all revisions. |

**Bulletproof Mechanisms**

- **Status Tracking**: Every analysis writes processing → complete or failed in the ingestion log. Survives container crashes.
- **Auto-Retry**: Retry Lambda runs every 15 minutes via CloudWatch Events. Finds tasks stuck in processing for >45 min or failed → re-launches ECS task.
- **Idempotency**: Lambda checks log before launching. ECS task updates status row at end. Exactly 1 row per analysis.
- **Each task processes its own analysis**: `specific_analysis_id` ensures no task picks up the wrong analysis.
- **Metadata for every analysis**: LLM tagger always runs — works with any JSON format.

**Supabase Migration**
- Created a safe migration script that only creates agent_* tables and webhook trigger
- Does NOT modify existing pipeline tables
- Added FK constraints on agent tables referencing the analyses table with ON DELETE CASCADE
- 9 existing analyses re-processed through the pipeline on the new database

**Docker Images**
- `halo-ingestion` — Ingestion pipeline (runs as one-off ECS tasks)
- `halo-query-plane` — Query plane API (runs as always-on ECS service)

**Final Data (after full pipeline run)**
- 8 analyses processed with complete data
- 8 metadata rows, 9,410 entities, 28,648 relationships, 280 communities
- 89 section chunks across all analyses

---

### Phase 5: RFP Data Pipeline & Production Hardening
*July 28 – 29, 2026*

Production deployment of the full ingestion pipeline with a real-world RFP dataset (~5.69 GB), compression pipeline, cross-project knowledge graph, and query plane with dynamic project detection.

**RFP Data Compression Pipeline (Jul 28)**

A 5.69 GB RFP submission folder (zip) was compressed to 270 MB (95% reduction) through a multi-stage pipeline:

1. **File-type stripping** — Removed 795 unwanted files (.dwg, .py, .jpeg, .json, .cfg, .exe, .bak, etc.) → 1.20 GB
2. **Image stripping** — Stripped 92,268 embedded images from PDFs, XLSX, and PPTX using PyMuPDF (fitz) → 895 MB
3. **Drawing deletion** — Removed 57 CAD drawing PDFs (floor plans, elevations, mark-ups — zero useful text) → 628 MB
4. **Office media cleanup** — Physically removed media files from inside Office zip archives (80 images from XLSX, 76 from PPTX) → 312 MB
5. **Font subsetting** — Ghostscript pdfwrite with font subsetting on all 322 PDFs → 270 MB

Each step verified for data integrity: text content compared word-for-word between original and compressed versions (100% match). All 59 XLSX verified for sheet and cell count preservation.

**Ingestion Pipeline Validated End-to-End**

Three test analyses processed through the full pipeline:

| Test | Files | Chunks | Entities | Relationships | Communities |
|---|---|---|---|---|---|
| Bahrain Bay Office | 1 | 13 | 247 | 1,000+ | 16 |
| KAFD Tower Block C | 1 | 14 | 353 | 3,388 | 14 |
| Yas Bay Boutique Hotel | 1 | 14 | 423 | 3,412 | 39 |

**Issues Identified & Fixed**

| Problem | Root Cause | Fix |
|---|---|---|
| ECS container failed to start | Docker image built for ARM64 (Apple Silicon) but ECS Fargate requires `linux/amd64` | Added `--platform linux/amd64` to all Docker builds |
| Query results truncated | PostgREST default limit of 1000 rows — queries without pagination silently dropped data | Added `fetch_all_paginated()` helper in config.py; updated 3 files (llm_relations.py, entity_extractor.py, chunk_linker.py) |
| File tracing capped at 100 | `source_b.py` used `.limit(100)` on analysis_files query | Increased to `.limit(500)` |
| Chunk content not searchable by project name | Query plane searches chunk content via ILIKE, but project name was only in metadata | Added `[Project: {name}]` prefix to every chunk's content in Source B |
| Query plane cache never expired | In-memory dict cache had no TTL — analysis was invisible until manual restart | Added 60-second TTL to all cache entries |
| New projects invisible to query planner | `detect_project_names()` only checked static alias list | Added dynamic fallback that extracts capitalized multi-word phrases from the question |
| "go no go" queries incorrectly filtered | Metadata retriever's risk detection set `decision=""no""` whenever "no-go" appeared in query | Added `and ""analysis"" not in lowered` guard — "go no go analysis" no longer triggers false filter |
| Project scope not tightened | `strengthen_metadata_filter_plan()` only updated scope when guardrails applied | Added fallback that tightens scope to `single_project` when project names are detected |
| ECS cached old image digest | `:latest` tag cached by ECS host — pushing new image didn't update running tasks | Registered new task definition revisions with explicit image tags (`:rev6`) |

**Fixes Deployed (rev 7)**

- `halo-ingestion` (ECS task): Rev 5 — pagination, .limit(500), [Project:] prefix
- `halo-query-plane` (ECS service): Rev 7 — dynamic project detection, 60s TTL cache, decision filter fix, project scope tightening
- Old revisions 1–6 deregistered to prevent rollback confusion

**Query Plane Improvements**

- Dynamic project name detection from natural language queries (no hardcoded alias list)
- 60-second cache TTL — new projects auto-discovered within 1 minute
- Decision filter no longer triggered by "go no go analysis" phrase
- Project scope correctly tightened to `single_project` for named-project queries
- All old task definitions cleaned up (rev 1–6 deregistered)

**Final System State**

- Ingestion pipeline: Fully automated, webhook-triggered, handles 50,000+ chunks with pagination
- Query plane: Always-on ECS service, auto-discovers new projects, 60s cache refresh
- 02_RFP.zip: 270 MB (from 5.69 GB original), text-verified, ready for pipeline ingestion
- All `agent_*` tables: Populated automatically end-to-end (chunks → entities → relationships → communities → metadata)

---

### Phase 6: Initial Pipeline Planning & n8n Bottleneck Analysis
*July 30, 2026*

**n8n Production Bottleneck Discovery**

Production monitoring of Sahil's n8n pipeline processing a large batch (analysis `db65adc0`, 369 files, 499 MB) revealed a critical bottleneck:

- n8n successfully chunked all 369 files into **1,215 chunks** (~546K words, avg 562 tokens per chunk)
- The embedding step failed with **HTTP 400: "maximum input length is 8192 tokens"** — 3 outlier chunks exceeded the 8K limit (largest: 12,570 tokens)
- n8n generates sections **one-by-one** (~2 min each) — the bottleneck is inherent to its sequential architecture
- Analysis reset and retried with a 2,000-word chunk cap to prevent recurrence

**Key findings for the Initial Pipeline design:**
- Chunk size enforcement is critical — a hard cap prevents embedding model failures
- 2,000-word chunks (~2,600 tokens) comfortably fit within any embedding model limit
- Our 2-call LLM approach (metadata + sections) is inherently faster than n8n's per-section generation
- DeepSeek V4 Flash's 64K+ token context eliminates the context window limitation entirely

**Initial Pipeline Plan Finalized**

Completed the full architecture plan to replace Sahil's n8n pipeline with a Python-based ECS Fargate pipeline:

- **5 phases**: Setup → Download/Parse/Extract → LLM Sections (2 calls) → Metadata → Finalize
- **Image extraction (Phase 2b)** — dual-output design:
  - **Step A**: OCR text from images → `analysis_chunks` (searchable alongside document text)
  - **Step B**: Image assets → `analysis_images` with full FK traceability (`chunk_id`, `source_file_id`, `analysis_id`, `organization_id`) + Bedrock Titan Multimodal embeddings (1024-dim)
- **3 storage buckets**: `rfp-uploads` (1 GB), `analysis-content` (100 MB), `analysis-images` (10 MB per image)
- **5 database tables**: analyses, analysis_files, analysis_chunks, analysis_sections, analysis_images
- **Independence**: No changes to existing Ingestion Pipeline or Query Plane

**Deloitte Competitive Analysis**

Completed comprehensive research on Deloitte's RFP AI pipeline tools (DocQMiner, NavigAite, Pursuit CoE, Formalyzer):

- Key gaps vs Halo RAG: **no knowledge graph**, **no image extraction**, **no vector embeddings**, **no community detection**
- Deloitte relies on human-in-the-loop validation — ours is fully automated
- DocQMiner uses traditional NLP ensembles (not GenAI) for extraction
- Documented in `deloitte-rfp-analysis-pipeline.md` as competitive reference

**Documents Updated (5 files in `~/Desktop/Sahil n8n replacement plan/`):**
- `initial-pipeline-plan.md` — Full plan with image extraction, DB tables, storage buckets
- `initial-pipeline-plan.docx` — Word document (master file)
- `supabase.md` — All schemas, FK traceability, storage buckets, row counts
- `halo-rag-current-state.md` — Current deployment state with image extraction notes
- `deloitte-rfp-analysis-pipeline.md` — New: 6-page competitive analysis

---

### Phase 7: Large-Batch Stress Test & Scalability Fixes
*July 31, 2026*

Proved the ingestion pipeline can process **5,000+ chunks** end-to-end by stress-testing it on the largest real-world batch yet, fixing three root-cause scalability bugs discovered along the way.

**The Big Batch (db65adc0)**
- 369 files, 498 MB (ZIPs, PDFs, XLSX, DOCX) → 4,694 chunks, 6 sections
- First time the pipeline handled a 5,000-chunk-scale analysis to completion

**3 Root-Cause Scalability Fixes**

| # | Bug | Symptom | Fix |
|---|---|---|---|
| 1 | `analysis_chunks` queries not paginated | PostgREST caps at 1,000 rows/query → silently dropped 3,694 of 4,694 chunks in Louvain + cross-chunk post-processor | Added `fetch_all_paginated()` to `graph/louvain.py` + `graph/cross_chunk_post_processor.py` |
| 2 | O(N×M) entity-resolution loop | For each of ~30K entity names, scanned all ~47K relationships (~1.4B ops) → froze for 14+ min, zero logs | Rebuilt as O(N+M): single-pass `name_meta` lookup dict then O(1) lookups (`graph/llm_relations.py`) |
| 3 | Single giant entity insert | One `insert()` of 30-60K rows exceeded Postgres statement timeout (57014) | Batch insert at 500 rows/insert, matching the relationship writer's pattern |

**Successful Result (rev11)**

| Metric | Value |
|---|---|
| Entities | 59,941 |
| Relationships | 106,483 |
| Communities | 28 |
| Section chunks | 11 |
| Total run time | ~63 min |
| Exit code | 0 (clean) |

**Other Operations (Jul 31)**
- **Query plane updated** — pulled Rayyanur's `query_plane/` from main (user-premise validation, resolved/unresolved conflict handling, stricter citation cleanup, community summary routing, clearer gap categories), built `halo-query-plane:rev10`, deployed to ECS
- **ALB idle timeout** raised 300s → **600s** so long relational queries don't get cut off
- **Docker image cleanup** — removed old rev9/rev10 ingestion + rev6/rev9 query-plane images from ECR and local
- **Relational query testing** — verified both agents answer multi-hop questions (parties↔payment terms, insurance↔bonds, disciplines↔subcontractors) with accurate citations across the 59K-entity graph

**Key takeaway:** The pipeline is now proven at 5,000-chunk scale — the pagination, loop-complexity, and insert-batching fixes eliminated all three classes of large-batch failure.

---

### Cloud & Infrastructure
| Technology | Usage |
|---|---|
| AWS ECS Fargate | Document pipeline compute (serverless containers) |
| AWS S3 | 6 buckets for raw/processed/artifact storage |
| AWS SQS | Job queue + dead-letter queue |
| AWS Lambda | S3 event trigger |
| AWS RDS PostgreSQL | Primary database with pgvector |
| AWS DynamoDB | Job tracking |
| AWS OpenSearch | Vector + text search |
| AWS Neptune | RDF/SPARQL graph database |
| AWS Bedrock | Embedding generation |
| Terraform | Infrastructure-as-code (35+ resources) |
| Docker | Containerization, multi-arch builds |
| CodeBuild | CI/CD build pipeline |

### Backend & Languages
| Technology | Usage |
|---|---|
| Python 3.12 | Primary language |
| FastAPI | API framework |
| PostgreSQL / pgvector | Relational + vector storage |
| NetworkX | Graph algorithms |
| scikit-learn | DBSCAN, Louvain community detection |
| pypdf / python-docx / openpyxl / python-pptx | Document parsing |

### LLM & AI
| Technology | Usage |
|---|---|
| OpenRouter | Unified LLM API |
| DeepSeek V4 Flash | Primary LLM |
| Gemini 2.5 Flash | Fallback LLM |
| Amazon Titan v2 | Text embeddings (1024-dim) |
| RAG | Hybrid search + LLM answer generation |
| GraphRAG | Knowledge-graph-enhanced retrieval |
| Prompt Engineering | Structured prompts for classification, tagging, arbitration |

### Frontend & Visualization
| Technology | Usage |
|---|---|
| D3.js (Canvas) | Force-directed graph visualization |
| HTML5 / CSS3 | Single-page application |
| FastAPI | Backend serving visualization data |

### DevOps & Tooling
| Technology | Usage |
|---|---|
| Git / GitHub | Version control |
| Supabase | PostgreSQL + pgvector managed service |
| Docker Compose | Local development environment |
| Model Context Protocol (MCP) | AI-assistant tool integration |
| Pytest | Unit testing |

---

## Architecture Patterns

### 1. Document Ingestion Pipeline (5-Stage Sequential)
```
Parse → Chunk → LLM Tag → Embed → Graph Extract
```
- SQS-based orchestration with continuous polling
- Auto post-processing after idle detection
- Dual LLM failover with token bucket rate limiting

### 2. RAG Query Pipeline (6-Path Parallel)
```
Query → LLM Classification → 6 Parallel Searches → BM25 Re-rank → Arbitration → Answer LLM
  ├─ Vector (pgvector)
  ├─ Keyword (FTS)
  ├─ Section
  ├─ Community (GraphRAG)
  ├─ Entity Graph
  └─ Metadata
```
- LLM-based query classification and project scope resolution
- Deterministic evidence arbitration
- Pre-LLM guard checks prevent off-topic answers

### 3. Knowledge Graph Explorer (Single-Page App)
```
Entity Search → Load Neighbourhood → Interactive Graph → Sidebar Details
```
- Tree-exploration growth (scroll-triggered loading)
- Stable layout during incremental expansion
- Global search across entities, documents, and chunk text

---

## Key Projects

| Project | Description | Approx. Commits |
|---|---|---|
| Ingestion Pipeline | Domain-agnostic document ingestion, 5 stages, AWS deployment, knowledge graph | ~80 commits |
| Graph Visualization | D3.js entity graph explorer with search, path finding, evidence panels | Part of pipeline |
| RAG Query Agent | Cross-project RAG with 6 search paths, deterministic arbitration | ~30 commits |
| Documentation Repo | Architecture docs, API references, security checklists, MCP guides | ~8 commits |
| Initial Pipeline Plan | n8n replacement design, image extraction pipeline, Deloitte competitive analysis | ~5 docs |
| Large-Batch Scaling | 4,694-chunk stress test, 3 scalability fixes (pagination, O(N+M) loop, batch insert) | rev11 image |

---

## Skills Demonstrated

- **Systems Architecture**: Designed and deployed production-grade multi-stage document processing pipeline on AWS with Terraform IaC
- **LLM Integration**: Multi-model failover, prompt engineering, batch processing, rate limiting
- **Database Design**: PostgreSQL with pgvector, full-text search, hybrid retrieval
- **Graph Algorithms**: NetworkX traversal, DBSCAN clustering, Louvain community detection, BFS path finding
- **Frontend Engineering**: D3.js force-directed graphs, Canvas rendering, event-driven UI
- **Infrastructure Engineering**: 35+ AWS resources, Docker multi-arch builds, CI/CD pipelines, IAM security
- **RAG Systems**: Query classification, evidence arbitration, guard checks, answer generation
- **Documentation**: Architecture docs, API references, security audits, deployment guides
- **Debugging**: Root-cause analysis identifying systemic bugs through trace-driven investigation
- **Production Monitoring**: Real-time pipeline monitoring, bottleneck identification, token limit diagnostics
- **Competitive Analysis**: Industry research comparing pipeline architectures against Deloitte enterprise tools
- **Technical Planning**: Architecture design for pipeline replacement with image extraction and multimodal search
- **Performance Engineering**: Algorithmic complexity fixes (O(N×M)→O(N+M)), query batching, pagination, large-batch debugging
