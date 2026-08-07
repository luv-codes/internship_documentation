# Internship Documentation — July 31 to August 7, 2026

> High-level overview of the internship work completed in this period.
> No proprietary code, client names, or confidential business logic is included.
> All descriptions are of technologies, architectures, and problem domains only.

---

## Overview

This period focused on taking the document-intelligence platform from a working
prototype to a **production-hardened, end-to-end system** capable of handling
**multi-gigabyte document sets** reliably. The work spanned four areas:

1. **Full end-to-end isolated deployment** — the complete three-plane pipeline
   (initial document ingestion, knowledge-graph ingestion, and query/retrieval)
   deployed to AWS as isolated infrastructure with a documented handoff runbook.

2. **Conversational query agents** — added session-based conversation memory to
   both query agents, with clarification/follow-up handling, stage-level logging,
   and a fast-path for casual/greeting messages that bypasses retrieval.

3. **Multi-gigabyte scalability hardening** — a systematic pass to remove every
   bottleneck that would fail on large uploads: model reliability, per-call
   timeouts, lossless compression budgets, parallel merges, and crash recovery.

4. **Frontend enablement & security** — authentication via organization
   membership, row-level-security policies, storage read access for results,
   and a large-upload trigger route.

---

## Timeline

### Phase 8: End-to-End Isolated Deployment (Jul 31 – Aug 1)
- Deployed the full three-plane pipeline (initial ingestion, document
  ingestion + knowledge graph, query) as **isolated AWS infrastructure** on a
  test account, with everything wired end-to-end.
- Wrote a **deep-detail deployment runbook** covering the handoff between
  planes and the SQS/Lambda/ECS orchestration.
- Added a **full setup README** so a fresh project can be brought up from
  scratch.

### Phase 9: Conversation Memory for Query Agents (Aug 1 – Aug 3)
- Added **session-based conversation memory** to the project-specific query
  agent: sessions are persisted, follow-up questions reuse context, and the
  resolver rewrites ambiguous references into standalone queries.
- Extended the same pattern to the **cross-project query agent**.
- Added **clarification handling** — when a user's question is ambiguous, the
  agent asks a clarifying question before running retrieval.
- Added **stage-level query logging** (session, resolver, memory-write, retrieval,
  answer timings) for observability.
- Added an optional **mini-agent fast-path**: greetings and casual day-to-day
  phrases are answered conversationally and never trigger the retrieval layer.

### Phase 10: Frontend Auth & Access Control (Aug 2 – Aug 4)
- Introduced **organization-membership-based authentication** across all query
  endpoints — the caller is identified by their membership row, from which the
  organization and user are derived (no mismatched org/user pairs possible).
- Added **row-level-security policies** enabling organization and member
  self-registration directly from the frontend.
- Added **strict per-organization isolation** for reference documents so no
  data leaks across organizations.
- Added **storage read policies** so the frontend can download processed images
  and structured section content for display.
- Added a **trigger route** that lets large uploads go straight to object
  storage (bypassing proxy size limits), then enqueue the document for
  processing via a lightweight JSON call.

### Phase 11: Multi-Gigabyte Scalability & Reliability (Aug 4 – Aug 6)
- **Model reliability**: migrated the section-building LLM calls to a direct
  model provider API, eliminating recurring proxy-side stalls and timeouts.
- **Per-call deadline**: added a total wall-clock timeout to every LLM call so
  a stalled response fails fast and retries, instead of hanging the pipeline
  indefinitely.
- **Lossless budgets**: sized digest, merge, and reduce token budgets so large
  batches are compressed losslessly in a single call — no truncation retries.
- **Parallel + resilient merges**: digest merging now runs in parallel, and a
  failed merge keeps the original content rather than dropping it.
- **Per-section isolation**: a single failing section no longer fails the whole
  document — it is rebuilt individually or falls back gracefully while the rest
  of the analysis completes.
- **Digest-map resilience**: one bad digest batch can never fail the entire
  analysis (extractive fallback preserves the content).
- **Crash-recovery watchdog**: an event-driven watchdog re-queues any analysis
  stuck in processing, so a dead worker self-heals.
- **OCR optimization**: skip OCR for pipeline-generated render images (whose
  source content is already extracted as text), while still OCR-ing every real
  embedded image.

### Phase 12: Cross-Project Conversational Routing & Polish (Aug 6 – Aug 7)
- Finalized the **cross-project conversational query agent**: it handles
  follow-ups, clarification, and conversational answers across multiple
  document sets.
- Added **conversational routing polish** — the agent distinguishes project-
  specific vs cross-project questions and routes accordingly.
- Verified the full platform end-to-end on production infrastructure:
  real frontend-triggered uploads flowed through initial ingestion →
  knowledge graph → query, with both agents answering from the results.
- Ran **content-presence verification** confirming extracted text, images,
  OCR, and captions match the source documents.

---

## Key Deliverables

- **Production-ready three-plane pipeline** on AWS (S3, SQS, Lambda, ECS
  Fargate, API Gateway / load balancers, CloudWatch).
- **Conversational query layer** with session persistence, clarification,
  follow-up resolution, and stage logging.
- **Multi-gigabyte-capable ingestion** — verified on real large document
  uploads with full content coverage.
- **Authentication & authorization** via organization membership with
  row-level-security isolation.
- **Comprehensive documentation**: deployment runbook, API reference, data
  architecture guide, and end-to-end setup guide.

---

## Technologies Used

- **AWS**: ECS Fargate, S3, SQS, Lambda, CloudWatch, IAM, load balancers.
- **Python**: FastAPI, PyMuPDF, openpyxl, python-docx, pytesseract, pandas,
  boto3, Pydantic, asyncio.
- **Databases**: PostgreSQL + pgvector, Supabase (auth, storage, row-level
  security, realtime).
- **LLMs**: large language models via direct provider APIs and proxy APIs,
  including vision-capable models for image understanding.
- **Search**: hybrid retrieval (vector + full-text + knowledge graph),
  deterministic evidence arbitration.
- **Frontend integration**: authenticated downloads of images and structured
  content, direct-to-storage uploads.

---

## Architecture Patterns

- **Event-driven pipeline**: S3/SQS triggers → Lambda dispatcher → ECS worker,
  with an event-driven watchdog for crash recovery.
- **Map-reduce summarization**: large document sets are digested in parallel,
  merged losslessly, then reduced into structured sections — bounded by token
  budgets that guarantee full coverage without truncation.
- **Session-based RAG**: conversation context persisted per user, with a
  resolver that rewrites follow-up questions and a fast-path for casual
  messages.
- **Strict tenant isolation**: every data access is scoped by organization at
  the database, storage, and API layers.

---

*Updated 7 August 2026.*
