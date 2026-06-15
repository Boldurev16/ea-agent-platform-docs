# 02 — PRD (AS-IS)

Документ описывает **реализованную** систему, не будущий продукт.  
Дата: `2026-06-07`. Метод: reverse-engineering из кода.

## 1. Product / System Name

**ea-agent-platform** — Corporate EA Architect Agent + RAG Platform.

## 2. Summary

Self-hosted платформа: ingest корпоративных документов → Qdrant; single agent «корпоративный архитектор» с JSON tool loop; primary API `POST /chat/agent` (SSE); legacy 4-role orchestrator deprecated.

**Evidence:** `README.md`, `app/api/main.py`, `orchestration/agent_stream.py`.

## 3. Problem Statement

| Problem | How addressed in code |
|---------|----------------------|
| EA knowledge scattered | `ingestion/pipeline.py`, `data/raw/` |
| Unreliable answers without sources | `retrieval/citations.py`, SSE `sources` |
| Complex multi-role orchestration | Single agent — `prompts/corporate_architect.py` |
| Mandatory pre-RAG latency | RAG on-demand — `search_kb` only in tool loop |

## 4. Target Users / Actors

| Actor | Interaction | Evidence |
|-------|-------------|----------|
| EA consultant (end user) | `/ui`, `POST /chat/agent` | `app/api/main.py` |
| KB operator | `data/raw`, `POST /ingestion/run`, `/kb/documents` | `app/api/main.py` |
| API integrator | REST + SSE | `11-api-and-integration-points.md` |
| Ops engineer | Docker, smoke scripts | `docker-compose.yml`, `scripts/smoke_*.ps1` |

## 5. Goals

### Business
- Консультации corporate EA function на русском с опорой на KB — `prompts/corporate_architect.py`
- Прозрачные citations — `Citation` model, SSE events

### Technical
- vLLM inference boundary — `llm/vllm_client.py`
- Nomic-only vector retrieval — `retrieval/similarity_search.py`
- Configurable infra providers — `core/provider_protocols.py` (M7)

## 6. In-Scope Capabilities (implemented)

### FR-1 Document ingestion
- **Requirement:** Parse PDF, XLSX, DOCX, PPTX, TXT from `data/raw/`
- **Evidence:** `ingestion/loaders/text_loader.py`, `test_ingestion_multiformat_loader.py`
- **Output:** chunks in Qdrant + `data/chunks.json`

### FR-2 Semantic retrieval
- **Requirement:** Query → embed → Qdrant cosine search → filter by threshold
- **Evidence:** `retrieval/similarity_search.py::search`
- **Default:** `top_k=5`, `similarity_threshold=0.25`

### FR-3 Agent tool loop
- **Requirement:** LLM returns JSON `action=tool|final`; execute tools; loop until final
- **Evidence:** `orchestration/agent_stream.py`, `parsers/agent_response.py`
- **Tools registered:** `search_kb` — `orchestration/agent_runtime.py::_default_registry`

### FR-4 Streaming chat
- **Requirement:** SSE events: status, tool_call, sources, text, introspect, close
- **Evidence:** `orchestration/agent_stream.py`, `app/api/sse.py::format_sse`

### FR-5 Session memory
- **Requirement:** Last N turns per `session_id` in prompt context
- **Evidence:** `memory/short_term.py`, max 10 turns
- **Optional Postgres:** `SESSION_STORE=postgres` → `memory/postgres_store.py`

### FR-6 Async ingest
- **Requirement:** `POST /ingestion/run` → job `queued`; worker → `done|failed`
- **Evidence:** `storage/ingest_jobs.py`, `ingestion/worker.py`, `GET /ingestion/jobs/{id}`

### FR-7 Health & readiness
- **Evidence:** `core/readiness.py`, endpoints `/health/live`, `/health/ready`, `/health/llm`

### FR-8 Legacy panel (deprecated)
- **Evidence:** `POST /tasks/panel`, `orchestration/orchestrator.py` with `legacy_deprecated`

## 7. Out-of-Scope / Not Implemented

| Item | Evidence of gap |
|------|-----------------|
| User authentication | No auth middleware in `app/api/main.py` |
| Workspaces UI | Single `CORPUS_ID_DEFAULT` |
| Confluence/SQL | `connectors/*/README.md` — "Not implemented" |
| MCP tools | Not in `tools/registry.py` |
| Embed widget | No `app/embed/` |
| Long-term semantic memory | Only short-term chat turns |
| BM25 hybrid | `retrieval/hybrid_retriever.py` aliases vector only |

## 8. Primary User Journeys

### Journey A — Chat with KB (primary)

1. User opens `/ui` or calls `POST /chat/agent` with `query`, `session_id`
2. `iter_agent_events` loads history from memory
3. LLM may call `search_kb` → Qdrant hits + citations
4. Final answer streamed; memory updated on `status=completed`

**Evidence:** `orchestration/agent_stream.py`, `app/api/main.py::chat_agent`

### Journey B — Operator ingest

1. Files in `data/raw/` OR paste via `/kb/documents`
2. `POST /ingestion/run` (async default) → `job_id`, `status=queued`
3. `ingestion/worker.py` claims job → `run_ingest_pipeline`
4. Poll `GET /ingestion/jobs/{job_id}` until `done`

**Evidence:** `storage/ingest_jobs.py`, `ingestion/worker.py`

### Journey C — Legacy multi-role panel

1. `POST /tasks/panel` → pre-RAG (`document_retrieve`) + 4 roles + nested `run_orchestration`
2. `legacy_deprecated: true` только внутри `response["orchestrator"]`, **не** на корне panel response

**Evidence:** `app/api/main.py::_build_agent_panel` (lines 80–125), `orchestration/orchestrator.py` (lines 107, 142)

## 9. Non-Functional Requirements (from code/config)

### Reliability
- Ingest job failure → `fail_ingest_job` with error text — `ingestion/worker.py`
- Qdrant down → `SimilaritySearchError` or empty collection response — `similarity_search.py`
- LLM unavailable → agent `status=error` — `agent_stream.py` lines 76–81
- **Needs verification:** no automatic retry on LLM calls in agent loop (double `generate` attempt only)

### Latency / Performance
- `LLM_REQUEST_TIMEOUT_SECONDS` default 8.0 — `runtime_settings.py` (`environment configuration template` uses 60 for agent)
- `AGENT_MAX_TOKENS` default 1024 — tool loop budget
- Qdrant batch upsert 100 points — `vectorstore/qdrant_store.py`
- **Inferred:** full re-ingest on each run (no delta checksum in pipeline)

### Scalability
- `SESSION_STORE=postgres` for multi-replica chat — `memory/short_term.py`
- Single worker polls ingest queue — **Needs verification:** throughput under concurrent jobs
- No horizontal agent sharding — in-memory sessions unless Postgres

### Security
- **No API authentication** — Confirmed gap
- MinIO/Qdrant/Postgres credentials via env — `environment configuration template`
- **Needs verification:** network isolation assumed via Docker compose

### Observability
- Health endpoints — `core/readiness.py`, `llm/runtime_health.py`
- Agent trace in SSE `introspect` event — `agent_stream.py`
- No structured metrics exporter (Prometheus) — **Confirmed gap**

### Maintainability
- Provider Protocols M7 — `core/provider_protocols.py`
- Smoke suite per milestone — `scripts/smoke_*.py`
- Pydantic DTOs — `core/contracts.py`

## 10. Dependencies & Constraints

| Dependency | Role | Config |
|------------|------|--------|
| vLLM | Chat LLM | `VLLM_BASE_URL`, port 8001 typical |
| LM Studio | Nomic embeddings | `LMSTUDIO_BASE_URL`, `LMSTUDIO_EMBED_MODEL` |
| Qdrant | Vector DB | `QDRANT_HOST`, `QDRANT_PORT` |
| Postgres | Jobs + optional sessions | `POSTGRES_*` |
| MinIO | Object storage sync | `STORAGE_PROVIDER=minio` |
| Docker | Runtime | `docker-compose.yml` |

**Constraint:** Embeddings через LM Studio, chat через vLLM — intentional split (`llm/lmstudio_embeddings.py`).

## 11. Risks & Limitations

| Risk | Severity | Evidence |
|------|----------|----------|
| No auth on API | High | `app/api/main.py` |
| LM Studio required for ingest/search | Medium | `embeddings/nomic_embeddings.py` |
| Full re-ingest cost | Medium | `run_ingest_pipeline` no delta |
| Legacy API surface confusion | Low | Multiple `/tasks/*` endpoints |
| `periodic-worker` restart loop | Low | compose — **Needs verification** |
| Local embed fallback in prod | Medium if enabled | `LMSTUDIO_EMBED_FALLBACK_ENABLED` |

## 12. Acceptance Criteria (current implementation)

| Criterion | Verification |
|-----------|--------------|
| Ingest produces Qdrant points = chunks | `smoke_ingest.ps1` |
| Retrieval returns citations | `smoke_retrieval.ps1` |
| Agent calls search_kb on KB question | `test_agent_runtime.py`, `smoke_agent_live.ps1` |
| SSE stream ends with `close` | `smoke_chat_agent.ps1` |
| Readiness reports dependencies | `smoke_m7.ps1` |
| Async ingest queued→done | `smoke_m7.py` |

## 13. Glossary

| Term | Meaning in this project |
|------|-------------------------|
| `corpus_id` | Logical KB namespace → Qdrant collection suffix |
| `search_kb` | Agent tool wrapping similarity search |
| Tool loop | LLM ↔ tools until `action=final` or limit |
| `session_id` | Chat session key for memory + SSE |
| Pre-RAG | Legacy: retrieval before orchestrator (deprecated) |
| SSE | Server-Sent Events for `/chat/agent` |
