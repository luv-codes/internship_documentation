# Internship Documentation — June 15 to July 22, 2026

> High-level overview of work completed during this internship period.
> No proprietary code, client names, or confidential business logic is included.
> All descriptions are of technologies, architectures, and problem domains only.

---

## Table of Contents

1. [Overview](#overview)
2. [Timeline](#timeline)
   - [Phase 1: Ingestion Pipeline & Knowledge Graph (Jun 15 – Jul 9)](#phase-1-ingestion-pipeline--knowledge-graph)
   - [Phase 2: Documentation & Handoff (Jul 7 – Jul 14)](#phase-2-documentation--handoff)
   - [Phase 3: RAG Query Agent (Jul 16 – Jul 22)](#phase-3-rag-query-agent)
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

### Phase 3: RAG Query Agent
*July 16 – 22, 2026*

Built a cross-project RAG query agent capable of answering questions across multiple document collections simultaneously.

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

---

## Technologies Used

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
