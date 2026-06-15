# 15 — Documentation Audit Report

Дата аудита: `2026-06-07`.  
Метод: сопоставление всех файлов `docs/generated/*.md` с кодом `ea-agent-platform` (исключая `.venv`).

---

## 1. Confirmed statements (выборка с evidence)

| # | Утверждение | Evidence |
|---|-------------|----------|
| C1 | Primary agent path: `iter_agent_events` → tool loop без pre-RAG | `orchestration/agent_stream.py`, `tools/builtin/search_kb.py` |
| C2 | `search_kb` вызывает только `perform_similarity_search`, не `retrieve_vector` | `tools/builtin/search_kb.py:15` |
| C3 | RAG on agent path: embed query → Qdrant `search_vectors` → threshold 0.25 | `retrieval/similarity_search.py:57-75`, `vectorstore/qdrant_store.py:89-110` |
| C4 | Nomic prefixes: `search_query:` / `search_document:` | `embeddings/nomic_embeddings.py:10-11, 21-31` |
| C5 | Embeddings HTTP через LM Studio adapter | `embeddings/nomic_embeddings.py` → `llm/lmstudio_embeddings.py` |
| C6 | Chat routing: `RUNTIME_PROVIDER=vllm` → `VLLMClient.chat_completion`; `tool` role → `[Tool result]` user message | `llm/runtime_router.py:20-31, 45-52` |
| C7 | Agent JSON steps: `action` ∈ `{tool, final}`; parse fallback на сырой текст | `parsers/agent_response.py:59-62`, `agent_stream.py:87-93` |
| C8 | `AGENT_MAX_TOOL_CALLS` default **5** | `runtime_settings.py:159` |
| C9 | Memory write только при `status==completed` + non-empty answer | `agent_stream.py:171-172` |
| C10 | `max_turns=10` in-memory и Postgres trim | `memory/short_term.py:17,30-31`, `postgres_store.py:53-60` |
| C11 | Postgres sessions: `SESSION_STORE=postgres` + healthy ping | `memory/short_term.py:46-53` |
| C12 | Ingest queue: `FOR UPDATE SKIP LOCKED` | `storage/ingest_jobs.py:93-101` |
| C13 | Job statuses Postgres: `queued` / `running` / `done` / `failed` | `storage/ingest_jobs.py`, `postgres_client.py` |
| C14 | Pipeline stages order в `run_ingest_pipeline` | `ingestion/pipeline.py:47-97` |
| C15 | Chunk `max_chars=220`, id `chk_` + SHA1[:12] | `ingestion/chunking/text_chunker.py:27-40` |
| C16 | Qdrant point id `uuid5(NAMESPACE_URL, chunk_id)` | `vectorstore/qdrant_store.py:30-31,64` |
| C17 | Qdrant upsert batch size **100** | `vectorstore/qdrant_store.py:76-79` |
| C18 | Collection name: `QDRANT_COLLECTION` для `default`; `{base}__{corpus}` для других | `runtime_settings.py:122-127` |
| C19 | Legacy pre-RAG: `document_retrieve` в orchestrator и role endpoints | `orchestration/orchestrator.py:54`, `main.py:725` |
| C20 | `legacy_deprecated: true` в ответе `run_orchestration` | `orchestration/orchestrator.py:107,142` |
| C21 | Local fallback `chunks.json`: только `retrieve_vector` при `SimilaritySearchError` + env | `retrieval/vector_retriever.py:72-75` |
| C22 | Readiness HTTP **200** даже при `status: degraded` | `app/api/main.py:591-594` |
| C23 | Compose: `STORAGE_PROVIDER=minio`, `SESSION_STORE=postgres`, `INGEST_ASYNC_DEFAULT=true` для app | `docker-compose.yml:59-63` |
| C24 | Worker command `python -m ingestion.worker`; poll **2.0s** | `docker-compose.yml:90`, `ingestion/worker.py:27` |
| C25 | KB snapshot <environment-specific-count> chunks / 9 sources в `data/ingestion_report.json` | `data/ingestion_report.json` (environment-specific) |
| C26 | `rerank_results` в `retrieval/reranker.py` **не вызывается** из hot path | grep project code (no imports except file itself) |
| C27 | `hybrid_retrieve` = alias `document_retrieve` | `retrieval/hybrid_retriever.py:8-9` |
| C28 | `graph_retriever.py` **не существует** в репозитории | glob search |

---

## 2. Corrected statements

| ID | Было в docs (неточность) | Коррекция | Evidence |
|----|--------------------------|-----------|----------|
| R1 | `legacy_deprecated` на `/tasks/panel` response root | Поле только внутри `response["orchestrator"]` от `run_orchestration`, не на корне panel | `main.py:121-125`, `orchestrator.py:107` |
| R2 | Mermaid agent flow: `search_kb` → Qdrant напрямую | Добавить `EmbeddingProvider.embed_query` между search и Qdrant | `similarity_search.py:58-59` |
| R3 | `POST /tasks/agent` «same agent logic» с session | `run_agent(payload.query)` **без** `session_id` — всегда `default` | `main.py:692-693`, `agent_runtime.py:26` |
| R4 | UI «Очистить историю» очищает memory | `clearHistory()` только `localStorage` (`dialogTurns`); **не** вызывает `memory.clear()` / API | `main.py:268-271` |
| R5 | UI history = server memory | Параллельно: server `session_id` (localStorage key `ea_agent_session_id_v1`) + отдельный UI `dialogTurns` | `main.py:207,366-379,540` |
| R6 | Periodic ingest копирует все inbox файлы | Только расширения: `.pptx`, `.pdf`, `.csv`, `.xlsx`, `.xls`, `.docx` | `periodic_ingest.py:10,28` |
| R7 | Periodic worker → pipeline напрямую | `run_periodic_once` → `_sync_inbox_to_raw` → `run_ingest()` → `run_ingest_pipeline()` | `periodic_ingest.py:41-45`, `run_ingest.py:8-9` |
| R8 | `docker-compose` app depends on all backends | `app` depends_on только **postgres** (healthy); qdrant/minio не в depends_on | `docker-compose.yml:69-71` |
| R9 | Worker depends postgres only | Worker: postgres (healthy) + qdrant (**started**, не healthy) | `docker-compose.yml:94-98` |
| R10 | LM Studio embed down → local fallback (generic) | На agent path (`search_kb`): `SimilaritySearchError`. Fallback только `retrieve_vector` / legacy paths | `vector_retriever.py:72-75` vs `search_kb.py` |
| R11 | `tests/test_health_endpoints.py` | Файл **не существует** | glob tests |
| R12 | Ingest JSONL status `completed` | Postgres API использует `done`; JSONL legacy entries могут иметь `completed` в старых строках | `ingest_jobs.jsonl` vs `complete_ingest_job` |
| R13 | `filter_identifiers` — exclude listed | **Confirmed:** `if identifier in request.filter_identifiers: continue` (skip hit) | `similarity_search.py:72-74` |
| R14 | Ingest reads MinIO for parsing | Parse path всегда `get_raw_documents_dir()` локально; MinIO только `sync_sources` upload | `ingestion/pipeline.py:37-51` |
| R15 | `LocalStorageProvider.sync_sources` no-op | Возвращает `[]`, не ошибка | `storage/providers.py:17-23` |
| R16 | Worker crash после claim | Job остаётся `running` — **нет** stale recovery в коде | `ingest_jobs.py` — no heartbeat |
| R17 | `GET /health/llm` module only `runtime_health.py` | Handler в `main.py`; импорт `check_runtime_health` из `llm` package (`llm/runtime_health.py`) | `main.py:596-617`, `llm/__init__.py:4` |
| R18 | `document_retriever` separate from similarity_search | Wrapper вокруг `perform_similarity_search` + string citations | `document_retriever.py:18-41` |
| R19 | 9 sources metric | Environment-specific snapshot; пересчитывается при каждом ingest | `ingestion_report.json` |
| R20 | Sequence diagram ingest: sync parallel branch | `sync_sources_to_storage` выполняется **до** `load_text_documents` в одном потоке | `pipeline.py:47-51` |

---

## 3. Unresolved ambiguities

| # | Вопрос | Статус | Notes |
|---|--------|--------|-------|
| U1 | Почему `ea-periodic-worker` restart loop | Needs verification | Нет qdrant в depends_on; ingest может fail на embed |
| U2 | Worker re-raise после `fail_ingest_job` — убивает процесс? | Needs verification | `worker.py:22-23`; compose `restart: unless-stopped` |
| U3 | Stale `running` jobs — ops procedure | Confirmed gap | No TTL/recovery |
| U4 | Max batch size `embed_texts` для <environment-specific-count> chunks | Needs verification | Single call `pipeline.py:67` |
| U5 | `LMSTUDIO_EMBED_FALLBACK_ENABLED` влияет на readiness vs runtime search | Needs verification | Fallback в `lmstudio_embeddings`, не в similarity_search |
| U6 | `blocking` readiness: embeddings `configured` vs `healthy` | Partially confirmed | `readiness.py:51` accepts both |
| U7 | vLLM вне compose — production networking | Assumption | `environment configuration template` <container-host-gateway> |
| U8 | Flaky `test_iter_agent_events_emits_sources_on_tool_call` | Needs verification | Qdrant-dependent |
| U9 | `ingest_jobs.jsonl` когда Postgres up — дублирование audit? | Needs verification | enqueue only Postgres when available |
| U10 | Panel relevance filter (token len ≥4) vs retrieval threshold | Documented gap | `main.py:85-109` — only panel, not agent |

---

## 4. Missing documentation areas

| Area | Gap | Suggested doc target |
|------|-----|----------------------|
| **UI dual history model** | Server memory vs `dialogTurns` localStorage vs `session_id` key | `05-memory.md`, `11-api` |
| **Legacy API surface** | `/tasks/role/*`, panel relevance filter, nested `legacy_deprecated` | `11-api`, `02-prd` |
| **`document_retriever` / `vector_retriever` / `hybrid_retriever`** | Legacy retrieval wrappers, fallback path | New section in `07-retrieval` |
| **4-role orchestration subgraphs** | `orchestration/roles/*`, planner, human_review | `04-agent-runtime` or legacy appendix |
| **`llm/lmstudio_embeddings` fallback** | `_local_fallback_embedding` when LM Studio down | `07-retrieval`, `09-ingestion` |
| **Storage provider resolution** | `auto` → minio if enabled else local | `storage/providers.py` |
| **MinIO upload contract** | `upload_object`, object_key pattern | `10-data-contracts` |
| **Retry policy scope** | `core/retry_policy.with_retries` — Qdrant ping only? | `12-ops` |
| **Agent panel `_build_user_answer`** | Extra user-facing answer layer in panel | Legacy docs |
| **No memory clear API** | `PostgresShortTermMemory.clear` exists but no HTTP | `05-memory`, `11-api` |
| **Periodic ingest limitations** | Extension whitelist, mtime-based copy | `09-ingestion-pipeline` |
| **Compose service dependency matrix** | Exact depends_on per service | `03-system-architecture`, `12-ops` |
| **JSONL vs Postgres job status vocabulary** | `done` vs `completed` | `10-data-contracts` |
| **Embed provider factory** | Only `nomic`/`lmstudio` alias | `embeddings/providers.py` |
| **Chat provider factory** | vllm path only in router | `llm/runtime_router.py` |
| **Tests map** | Which smoke covers which path | `12-ops` |
| **Deleted `graph_retriever`** | Stale references in non-generated docs | `docs/2026-05-31_langchain-core-c4-layers.md` |

---

## Mermaid diagram verification

| Diagram | Location | Verdict | Fix |
|---------|----------|---------|-----|
| User → agent → Qdrant | `03-system-architecture.md` §3.1 | **Incomplete** | Missing `EmbeddingProvider.embed_query` |
| Ingest pipeline flowchart | `03`, `09` | **OK** | sync_minio optional first step — correct order |
| Memory stateDiagram | `03`, `05` | **OK** | Matches read/write gate |
| Worker job lifecycle | `08`, `14` | **OK** | Status names correct for Postgres |
| search_kb sequence | `06`, `14` | **Incomplete** | Missing embed step |
| Cross-area map | `14` | **OK** | High-level accurate |
| Agent runtime stateDiagram | `04` | **OK** | Matches `iter_agent_events` |
| Executive flowchart | `01` | **OK** | Optional search_kb correct |

---

## Applied corrections (this audit)

Следующие generated files обновлены по результатам аудита:

- `02-prd.md` — Journey C `legacy_deprecated` nesting
- `03-system-architecture.md` — sequence diagram + compose depends_on
- `04-agent-runtime.md` — `/tasks/agent` session_id gap
- `05-memory.md` — UI vs server memory
- `09-ingestion-pipeline.md` — periodic extensions
- `11-api-and-integration-points.md` — legacy flags, tasks/agent, memory clear
- `12-ops-observability-and-risks.md` — compose deps, test reference
- `README.md` — link to this report

---

## Audit confidence legend

- **Confirmed** — прямое соответствие коду
- **Corrected** — было неточно в docs, исправлено
- **Needs verification** — требует runtime/staging проверки
- **Environment-specific** — зависит от `data/` snapshot, не от кода
