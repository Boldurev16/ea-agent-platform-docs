# Runbook: Cross-area map (chat + ingest)

**Схема:** [14-deep-dive-priority-areas.md](../14-deep-dive-priority-areas.md) — Cross-area interaction map.  
**Audit:** [15-documentation-audit-report.md](../15-documentation-audit-report.md).  
**Шпаргалка:** [14-docs-cheat-sheet-ru.md](../14-docs-cheat-sheet-ru.md).

---

## Как читать схему за 2 минуты

1. **Два независимых потока** на одной картинке: сверху — **запрос пользователя** (chat), снизу — **индексация** (ingest). Они связаны только через **Qdrant** и **Embed** (LM Studio + Nomic).
2. **Chat не индексирует.** Вопрос в UI не запускает pipeline — если KB пустая, `search_kb` не поможет без ingest.
3. **`Retrieval`** на схеме = модуль `similarity_search` (не отдельный сервис). **`Embed`** = `EmbeddingProvider` → HTTP LM Studio.
4. **`Memory`** — только server-side turns по `session_id`; UI `dialogTurns` на схеме **не показан** (audit R5).
5. **Стрелки** — направление control/data flow; optional tool path = LLM может ответить без `search_kb`.
6. При инциденте: определите **который поток сломан** (ответы vs индексация), затем идите по таблице «Что может пойти не так» ниже.

---

## Основной путь запроса (query path)

End-to-end от HTTP до ответа. Primary API: `POST /chat/agent` (SSE).

| Шаг | Участок | Код / endpoint | Диаграмма | Документ |
|-----|---------|----------------|-----------|----------|
| 1 | HTTP entry | `app/api/main.py::chat_agent` | [agent-runtime-sequence](agent-runtime-sequence-legend-ru.md) | [11-api](../11-api-and-integration-points.md) |
| 2 | Memory read | `get_short_term_memory().get_history(session_id)` | [memory-lifecycle](memory-lifecycle-legend-ru.md) | [05-memory](../05-memory.md) |
| 3 | Prompt + loop | `iter_agent_events` → `_build_messages` | [agent-runtime-state](agent-runtime-state-legend-ru.md) | [04-agent-runtime](../04-agent-runtime.md) |
| 4 | LLM chat | `generate_chat_completion` → vLLM (typical) | [agent-runtime-sequence](agent-runtime-sequence-legend-ru.md) | [03-system-architecture](../03-system-architecture.md) §3.1 |
| 5 | Parse step | `parse_agent_step` → `tool` \| `final` | [agent-tool-loop](agent-tool-loop-legend-ru.md) | [04-agent-runtime](../04-agent-runtime.md) |
| 6a | *(optional)* Tool | `ToolRegistry` → `search_kb` | [search-kb-sequence](search-kb-sequence-legend-ru.md) | [06-tools-search_kb](../06-tools-search_kb.md) |
| 6b | *(optional)* Retrieval | `perform_similarity_search` → `embed_query` → Qdrant | [retrieval-similarity-search](retrieval-similarity-search-legend-ru.md) | [07-retrieval](../07-retrieval-similarity_search.md) |
| 7 | Tool result → LLM | JSON в `messages` role `tool` | [agent-tool-loop](agent-tool-loop-legend-ru.md) | [04-agent-runtime](../04-agent-runtime.md) |
| 8 | Final + SSE | `text`, `sources`, `close` events | [agent-runtime-sequence](agent-runtime-sequence-legend-ru.md) | [11-api](../11-api-and-integration-points.md) |
| 9 | Memory write | `append_turn` только при `status==completed` | [memory-lifecycle](memory-lifecycle-legend-ru.md) | audit C9 |

**Сводная строка (audit C1):** User → `/chat/agent` → memory read → LLM → *(optional)* `search_kb` → `embed_query` → Qdrant → LLM → memory write.

**Не на этом пути:** `POST /tasks/agent` — sync, **без** `session_id` (всегда `default`, audit R3). Legacy pre-RAG: `/tasks/orchestrate`, `/tasks/panel`, `/tasks/role/*` через `document_retrieve` (audit C19).

**Быстрые проверки:** `GET /health/llm`, `GET /health/ready`, smoke `scripts/smoke_chat_agent.ps1`.

---

## Основной путь ingestion (index path)

End-to-end от файлов до Qdrant. Trigger: `POST /ingestion/run` (async default) или worker/periodic.

| Шаг | Участок | Код | Диаграмма | Документ |
|-----|---------|-----|-----------|----------|
| 1 | Enqueue | `enqueue_ingest_job` → Postgres `ingest_jobs` `queued` | [worker-job-states](worker-job-states-legend-ru.md) | [08-ingestion-worker](../08-ingestion-worker.md) |
| 2 | Claim | `claim_next_ingest_job` (`FOR UPDATE SKIP LOCKED`) → `running` | [worker-job-states](worker-job-states-legend-ru.md) | audit C12 |
| 3 | Worker | `ingestion/worker.py::process_next_job` | [worker-job-states](worker-job-states-legend-ru.md) | [08-ingestion-worker](../08-ingestion-worker.md) |
| 4 | Optional MinIO sync | `sync_sources_to_storage` (`STORAGE_PROVIDER=minio`) | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | audit R14 |
| 5 | Источник разбора | Локальный каталог исходных документов; MinIO здесь не является read-path | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | [09-ingestion-pipeline](../09-ingestion-pipeline.md) |
| 6 | ETL | load → parse → normalize → `chunk_records(220)` | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | audit C15 |
| 7 | Embed batch | `embed_texts` (Nomic `search_document:`) | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | audit C5, U4 |
| 8 | Qdrant upsert | `upsert_embeddings` batches of 100 | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | audit C16–C17 |
| 9 | Job complete | `complete_ingest_job` → status `done` + report | [worker-job-states](worker-job-states-legend-ru.md) | audit C13, R12 |
| 10 | Artifacts | `ingestion_report.json`, `chunks.json` | [ingestion-pipeline](ingestion-pipeline-legend-ru.md) | [09-ingestion-pipeline](../09-ingestion-pipeline.md) |

**Сводная строка:** оператор → `POST /ingestion/run` → `ingest_jobs` → worker → `run_ingest_pipeline` → исходные документы → embeddings → Qdrant → отчет.

**Периодический путь (отдельно):** periodic worker → входной каталог → поддержанные форматы документов → `run_ingest()` → pipeline.

**Быстрые проверки:** `GET /ingestion/jobs/{job_id}`, `data/ingestion_report.json`, smoke `scripts/smoke_ingest.ps1`, `scripts/smoke_m7.ps1`.

---

## Что может пойти не так на каждом участке

### Query path

| Участок | Симптом | Likely cause (confirmed) | Действие | Docs | Audit |
|---------|---------|--------------------------|----------|------|-------|
| **HTTP / SSE** | Нет stream, 4xx | Невалидный body | Проверить `ChatAgentRequest` | [11-api](../11-api-and-integration-points.md) | — |
| **Memory read** | Агент «не помнит» | Другой `session_id`; или UI `dialogTurns` ≠ server memory | Сравнить `session_id` в request vs `ea_agent_session_id_v1` | [05-memory](../05-memory.md) | R5 |
| **Memory read** | Пустая история | Новая сессия; `POST /tasks/agent` всегда `default` | Использовать `/chat/agent` + `session_id` | [04-agent-runtime](../04-agent-runtime.md) | R3 |
| **LLM (vLLM)** | `status=error`, hint про `/health/llm` | vLLM down / unreachable | `GET /health/llm`; проверить `RUNTIME_PROVIDER`, `VLLM_BASE_URL` | [12-ops](../12-ops-observability-and-risks.md) | C6, U7 |
| **Parse** | `parse_fallback`, сырой текст | LLM не вернул JSON | Prompt / model behavior | [04-agent-runtime](../04-agent-runtime.md) | C7 |
| **Tool loop** | `status=max_rounds` | `AGENT_MAX_TOOL_CALLS` (default 5) | SSE `introspect`; dedupe / repeated tools | [04-agent-runtime](../04-agent-runtime.md) | C8 |
| **Tool dedupe** | «Duplicate tool call blocked» | `ToolCallDeduper` | Ожидаемо; смотреть `introspect` | [04-agent-runtime](../04-agent-runtime.md) | — |
| **search_kb** | `{error}` в tool result | Unknown tool / exception | Trace `result_preview` | [06-tools-search_kb](../06-tools-search_kb.md) | — |
| **embed_query** | Tool error, search fails | LM Studio embed down | `GET /health/ready` → embeddings; **не** local chunks fallback на agent path | [07-retrieval](../07-retrieval-similarity_search.md) | R10, C21, U5 |
| **Qdrant search** | Пустые hits / error | Collection missing, Qdrant down, threshold 0.25 | `GET /health/ready`; `data/chunks.json`; ingest report | [07-retrieval](../07-retrieval-similarity_search.md) | C3 |
| **Memory write** | История не сохранилась | `status!=completed` или пустой answer | Норма для error/max_rounds | [05-memory](../05-memory.md) | C9 |
| **UI clear history** | Clear не помогает агенту | `clearHistory()` только localStorage | Новый `session_id`; нет HTTP clear API | [11-api](../11-api-and-integration-points.md) | R4, gap |

### Index path

| Участок | Симптом | Likely cause (confirmed) | Действие | Docs | Audit |
|---------|---------|--------------------------|----------|------|-------|
| **Enqueue** | Job не создаётся | Postgres down → JSONL only; worker не читает JSONL | Postgres healthy; `ea-postgres` | [08-ingestion-worker](../08-ingestion-worker.md) | — |
| **Claim / stale `running`** | Job вечно `running` | Worker crash после claim; **нет recovery** | Postgres `ingest_jobs`; restart worker; manual fix | [08-ingestion-worker](../08-ingestion-worker.md) | U3, R16 |
| **Worker process** | `ea-worker` restart loop | Pipeline exception → `fail_ingest_job` + re-raise | Логи worker; `error` в job row | [08-ingestion-worker](../08-ingestion-worker.md) | U2 |
| **Нет документов для разбора** | Job `failed` | Пустой или неподдерживаемый источник документов | Проверить skipped report и добавить файлы | [09-ingestion-pipeline](../09-ingestion-pipeline.md) | C14 |
| **MinIO sync** | Sync empty | `LocalStorageProvider` no-op; MinIO not configured | `STORAGE_PROVIDER`; optional step | [09-ingestion-pipeline](../09-ingestion-pipeline.md) | R14, R15 |
| **embed_texts** | Ingest fail / slow | LM Studio down; huge single batch | `LMSTUDIO_EMBED_FALLBACK_ENABLED`; embed health | [09-ingestion-pipeline](../09-ingestion-pipeline.md) | U4, U5 |
| **Qdrant upsert** | Job `failed` | Qdrant unreachable | `ea-qdrant`; worker depends qdrant started | [12-ops](../12-ops-observability-and-risks.md) | R9 |
| **Job status API** | `completed` vs `done` | JSONL legacy vs Postgres `done` | Poll until `done` \| `failed` | [10-data-contracts](../10-data-contracts-and-models.md) | R12 |
| **Periodic worker** | Restart loop, файлы не попадают в обработку | Поддерживается ограниченный набор расширений | Проверить правила periodic ingest | [09-ingestion-pipeline](../09-ingestion-pipeline.md) | U1, R6 |
| **Re-ingest** | Долго каждый раз | Full reprocess, no delta | Ожидаемо по коду | [09-ingestion-pipeline](../09-ingestion-pipeline.md) | C14 |

### Shared (Qdrant + Embed)

| Участок | Симптом | Действие | Docs | Audit |
|---------|---------|----------|------|-------|
| **Readiness degraded** | API 200 но `status: degraded` | `GET /health/ready` → `blocking` | [12-ops](../12-ops-observability-and-risks.md) | C22, U6 |
| **Embed fallback** | «Успешный» ingest, плоский search | Fallback в `lmstudio_embeddings`, не similarity_search | Проверить env в compose | [07-retrieval](../07-retrieval-similarity_search.md) | U5, R10 |
| **Legacy retrieval** | Разные hits в panel vs agent | Panel: pre-RAG + token filter; agent: on-demand tool | Не смешивать endpoints | [11-api](../11-api-and-integration-points.md) | U10, C19 |

---

## Узлы схемы (reference)

| Узел | Процесс / модуль |
|------|------------------|
| `User` | Chat client |
| `API` | FastAPI `ea-app` (`app/api/main.py`) |
| `iter_agent_events` | `orchestration/agent_stream.py` |
| `Memory` | `memory/short_term.py`, `postgres_store.py` |
| `LLM` | vLLM via `llm/runtime_router.py` (typical) |
| `search_kb` | `tools/builtin/search_kb.py` |
| `similarity_search` | `retrieval/similarity_search.py` |
| `Qdrant` | `vectorstore/qdrant_store.py` |
| `Embed` | `embeddings/nomic_embeddings.py` → LM Studio |
| `Operator` | KB operator |
| `POST /ingestion/run` | `app/api/main.py::ingestion_run` |
| `ingest_jobs` | `storage/ingest_jobs.py` |
| `Worker` | `ingestion/worker.py` |
| `run_ingest_pipeline` | `ingestion/pipeline.py` |

## Глоссарий меток (E2E)

| Label | Кратко RU | Путь |
|-------|-----------|------|
| `runtime` | Agent orchestration | Chat |
| `pipeline` | Ingest ETL | Indexing |
| `worker` | Job consumer | Async ingest |
| `queue` / `ingest_jobs` | Postgres job table | API ↔ worker |
| `vector store` / `Qdrant` | Shared index | Both paths |
| `chunk(220)` | Indexed text unit | Ingest only |
| `session_id` | Server memory key | Chat only (not `dialogTurns`) |
| `legacy_deprecated` | Flag в `run_orchestration` | Legacy only; not panel root (R1) |

## Типичные ошибки понимания (audit-corrected)

- Ingest **не** запускается из chat.
- Chat RAG **не** читает MinIO — search идёт в Qdrant после `embed_query`.
- Agent path **без** local `chunks.json` fallback (R10).
- `app` compose depends_on **postgres only** — не все backends (R8).

## Связанные документы

- [03-system-architecture.md](../03-system-architecture.md)
- [14-deep-dive-priority-areas.md](../14-deep-dive-priority-areas.md)
- [15-diagram-legends-ru.md](../15-diagram-legends-ru.md)
- [13-open-questions.md](../13-open-questions.md)
