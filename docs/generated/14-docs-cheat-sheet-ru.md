# Шпаргалка по документации ea-agent-platform

Быстрый навигатор для onboarding. Язык: русский; идентификаторы кода — в оригинале.  
Связанные файлы: [README](README.md), [audit report](15-documentation-audit-report.md), [легенды диаграмм](15-diagram-legends-ru.md).

---

## 1. Карта документов

| Файл | Что внутри | Когда читать | Для кого |
|------|------------|--------------|----------|
| [README.md](README.md) | Хаб, порядок чтения по ролям, легенда достоверности | Первый вход в `docs/generated/` | Все |
| [Publishing & scope](../publishing-sanitization-report.md) | Public export boundaries |
| [01-executive-overview.md](01-executive-overview.md) | Executive summary, in/out of scope, снимок KB | Старт для stakeholders | Lead, PM, architect |
| [02-prd.md](02-prd.md) | PRD AS-IS, user journeys, NFR из кода | Понять «что продукт делает сейчас» | Product, architect |
| [03-system-architecture.md](03-system-architecture.md) | Контекст, контейнеры, основные Mermaid flows, compose deps | Общая картина системы | Architect, backend |
| [04-agent-runtime.md](04-agent-runtime.md) | `iter_agent_events`, tool loop, SSE, ошибки | Дебаг агента, интеграция chat | Backend, AI |
| [05-memory.md](05-memory.md) | Session memory, Postgres vs in-memory, **UI vs server** | Странная история диалога | Backend, frontend |
| [06-tools-search_kb.md](06-tools-search_kb.md) | Tool `search_kb`, I/O, call chain | Плохие hits в агенте | Backend, AI |
| [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md) | `perform_similarity_search`, threshold, Qdrant | Настройка retrieval, пустые hits | ML, backend |
| [08-ingestion-worker.md](08-ingestion-worker.md) | Async worker, `ingest_jobs`, poll loop | Зависшие/failed jobs | Backend, ops |
| [09-ingestion-pipeline.md](09-ingestion-pipeline.md) | ETL stages, chunk 220, periodic ingest | Индексация, re-ingest | Backend, data |
| [10-data-contracts-and-models.md](10-data-contracts-and-models.md) | DTOs, Postgres tables, env vars | Интеграция, контракты API | Все engineers |
| [11-api-and-integration-points.md](11-api-and-integration-points.md) | Endpoints, SSE, legacy API | Подключение клиента | Integrators, frontend |
| [12-ops-observability-and-risks.md](12-ops-observability-and-risks.md) | Compose, smoke, risk register | Деплой, on-call | SRE, ops |
| [13-open-questions.md](13-open-questions.md) | Backlog неясностей | Планирование, архитектурное ревью | Architect, lead |
| [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) | Глубокий разбор 6 критических путей | После architecture, перед кодом | Senior backend |
| [15-documentation-audit-report.md](15-documentation-audit-report.md) | Confirmed / corrected / gaps после ревью кода | Проверка «что точно верно» | Architect, reviewer |
| [15-diagram-legends-ru.md](15-diagram-legends-ru.md) | Индекс Mermaid + как читать диаграммы | Непонятна схема в docs | Все |
| [14-docs-cheat-sheet-ru.md](14-docs-cheat-sheet-ru.md) | Этот файл | Ежедневная навигация | Все |

---

## 2. Куда смотреть, если…

### `search_kb` возвращает не те результаты

**Симптом:** агент цитирует не тот документ, пустые hits, низкий score.

| Действие | Документы |
|----------|-----------|
| Понять chain tool → retrieval | [06-tools-search_kb](06-tools-search_kb.md), [07-retrieval-similarity_search](07-retrieval-similarity_search.md) |
| Проверить индекс и chunks | [09-ingestion-pipeline](09-ingestion-pipeline.md), `data/chunks.json`, `data/ingestion_report.json` |
| Legacy path (не agent) | [07](07-retrieval-similarity_search.md) — `document_retriever`, `vector_retriever` |

**Audit:** C2, C3, R10 (agent path без local fallback). **Ambiguity:** U10 (panel filter ≠ agent).

**Код:** `tools/builtin/search_kb.py`, `retrieval/similarity_search.py`, `similarity_threshold=0.25`, `top_k`.

---

### `similarity_search` — мало или слишком много chunks

**Симптом:** `filtered_hits=0` в debug или слишком широкий контекст.

| Действие | Документы |
|----------|-----------|
| Параметры threshold / top_k | [07-retrieval-similarity_search](07-retrieval-similarity_search.md) |
| Как chunks создаются (220 chars) | [09-ingestion-pipeline](09-ingestion-pipeline.md) |
| Контракты | [10-data-contracts-and-models](10-data-contracts-and-models.md) |

**Audit:** C3, C15, R13 (`filter_identifiers` исключает listed). **Gap:** `RetrievalRequest.filters` не применяются в `search()`.

**Код:** `retrieval/similarity_search.py:69-75`, env `CORPUS_ID_DEFAULT`.

---

### Agent runtime странно (много tool calls, нет `sources`, падения)

**Симптом:** `status=max_rounds`, нет SSE `sources`, `status=error`, `parse_fallback`.

| Действие | Документы |
|----------|-----------|
| Execution loop | [04-agent-runtime](04-agent-runtime.md), [14-deep-dive](14-deep-dive-priority-areas.md) §1 |
| API / SSE events | [11-api-and-integration-points](11-api-and-integration-points.md) |
| LLM health | [12-ops](12-ops-observability-and-risks.md) — `/health/llm` |

**Audit:** C7, C8 (`AGENT_MAX_TOOL_CALLS=5`), R3 (`POST /tasks/agent` без `session_id`).

**Код:** `orchestration/agent_stream.py`, `parsers/agent_response.py`, `ToolCallDeduper`, `GET /health/llm`.

---

### Ingestion зависает или падает (worker / pipeline)

**Симптом:** job `failed`, worker restart, долгий ingest.

| Действие | Документы |
|----------|-----------|
| Worker + queue | [08-ingestion-worker](08-ingestion-worker.md) |
| Pipeline stages | [09-ingestion-pipeline](09-ingestion-pipeline.md) |
| Ops / compose | [12-ops](12-ops-observability-and-risks.md), [03-system-architecture](03-system-architecture.md) |

**Audit:** C12–C17, R14 (parse из `data/raw/`, не MinIO), U4 (batch embed size).

**Код:** `ingestion/worker.py`, `ingestion/pipeline.py`, `GET /ingestion/jobs/{job_id}`.

---

### Двойная история (server memory vs `dialogTurns`) ведёт себя неожиданно

**Симптом:** UI показывает историю, агент «не помнит»; или наоборот.

| Действие | Документы |
|----------|-----------|
| Server memory | [05-memory](05-memory.md) |
| API session | [11-api](11-api-and-integration-points.md) — только `POST /chat/agent` + `session_id` |

**Audit:** R4, R5 — **два независимых** хранилища. UI `dialogTurns` ≠ `chat_turns` / `ShortTermMemory`.

**Код:** `memory/short_term.py`, `app/api/main.py` (JS: `ea_agent_session_id_v1`, `dialogTurns`).

---

### Clear history в UI не очищает серверную память

**Симптом:** после «Очистить историю» агент всё ещё учитывает прошлые turns.

**Это ожидаемое поведение по коду:** `clearHistory()` только `localStorage`.

| Действие | Документы |
|----------|-----------|
| Объяснение | [05-memory](05-memory.md), [11-api](11-api-and-integration-points.md) |
| Workaround | Новый `session_id` в `POST /chat/agent` или прямой SQL/API clear (**API clear не реализован**) |

**Audit:** R4, missing doc «No memory clear API».

---

### Периодический ingest ведёт себя странно

**Симптом:** файлы из inbox не попали в raw; worker restart loop.

| Действие | Документы |
|----------|-----------|
| Periodic limits | [09-ingestion-pipeline](09-ingestion-pipeline.md) — только 6 расширений |
| Compose deps | [03-system-architecture](03-system-architecture.md) — periodic без qdrant |

**Audit:** R6, R7, U1. **Код:** `ingestion/periodic_ingest.py`, `TRACKED_EXTENSIONS`.

---

### `LMSTUDIO_EMBED_FALLBACK_ENABLED` ломает ingest или search

**Симптом:** ingest «успешен» с фейковыми векторами; search даёт странные scores.

| Действие | Документы |
|----------|-----------|
| Agent path vs legacy | [07-retrieval](07-retrieval-similarity_search.md), [15-audit](15-documentation-audit-report.md) R10 |
| Ops env | [12-ops](12-ops-observability-and-risks.md), compose `LMSTUDIO_EMBED_FALLBACK_ENABLED` |

**Audit:** U5 — влияние на readiness vs runtime **требует проверки**. Fallback в `llm/lmstudio_embeddings.py`, **не** в `similarity_search`.

**Код:** `llm/lmstudio_embeddings.py`, `embeddings/nomic_embeddings.py`.

---

### Stale jobs застревают в `running`

**Симптом:** `GET /ingestion/jobs/{id}` → `status=running` долго после crash worker.

| Действие | Документы |
|----------|-----------|
| Job lifecycle | [08-ingestion-worker](08-ingestion-worker.md) |
| Gaps | [13-open-questions](13-open-questions.md), audit U2, U3 |

**Confirmed gap:** нет stale recovery / TTL в коде. **Ручная проверка:** Postgres `ingest_jobs`, логи `ea-worker`.

---

## 3. Типичные маршруты чтения

### Архитектор

1. [01-executive-overview](01-executive-overview.md) — границы системы  
2. [03-system-architecture](03-system-architecture.md) — контекст + flows  
3. [15-documentation-audit-report](15-documentation-audit-report.md) — что точно подтверждено  
4. [10-data-contracts-and-models](10-data-contracts-and-models.md) — контракты  
5. [13-open-questions](13-open-questions.md) — backlog рисков  

### Backend engineer

1. [14-docs-cheat-sheet-ru](14-docs-cheat-sheet-ru.md) (этот файл)  
2. [11-api-and-integration-points](11-api-and-integration-points.md)  
3. [04-agent-runtime](04-agent-runtime.md)  
4. [06-tools-search_kb](06-tools-search_kb.md) + [07-retrieval-similarity_search](07-retrieval-similarity_search.md)  
5. [08-ingestion-worker](08-ingestion-worker.md) + [09-ingestion-pipeline](09-ingestion-pipeline.md)  
6. [05-memory](05-memory.md) — до правок UI  

### ML / AI engineer

1. [07-retrieval-similarity_search](07-retrieval-similarity_search.md)  
2. [06-tools-search_kb](06-tools-search_kb.md)  
3. [09-ingestion-pipeline](09-ingestion-pipeline.md) — chunking, embed  
4. [04-agent-runtime](04-agent-runtime.md) — prompts, tool loop  
5. [10-data-contracts-and-models](10-data-contracts-and-models.md) — `RetrievalResponse`, env  

### Support / ops engineer

1. [12-ops-observability-and-risks](12-ops-observability-and-risks.md)  
2. [11-api](11-api-and-integration-points.md) — health endpoints  
3. [08-ingestion-worker](08-ingestion-worker.md)  
4. [09-ingestion-pipeline](09-ingestion-pipeline.md)  
5. [15-diagram-legends-ru](15-diagram-legends-ru.md) — cross-area map  

### Новый разработчик (junior/mid)

1. [README](README.md)  
2. [01-executive-overview](01-executive-overview.md)  
3. [03-system-architecture](03-system-architecture.md) + [15-diagram-legends-ru](15-diagram-legends-ru.md)  
4. [11-api](11-api-and-integration-points.md) — `POST /chat/agent`  
5. [04-agent-runtime](04-agent-runtime.md)  
6. [14-deep-dive-priority-areas](14-deep-dive-priority-areas.md) — при первом дебаге  

---

## 4. Топ‑20 терминов

| Термин | Кратко |
|--------|--------|
| **agent runtime** | Оркестрация tool loop вокруг LLM; главный код — `iter_agent_events` |
| **`iter_agent_events`** | Generator SSE-событий и sync-логики агента |
| **`run_agent`** | Sync wrapper; собирает `close` из iterator |
| **`ToolCallDeduper`** | Блокирует повторный tool call с теми же args в одном запросе |
| **`AGENT_MAX_TOOL_CALLS`** | Env, default **5** — лимит tool rounds |
| **server memory** | `get_short_term_memory()` → turns по `session_id` (in-mem или Postgres) |
| **`dialogTurns`** | UI localStorage для отображения; **не** server memory |
| **`session_id`** | Ключ server memory; в UI — `ea_agent_session_id_v1` |
| **`search_kb`** | Builtin tool агента; on-demand RAG |
| **`perform_similarity_search`** | Core retrieval; embed + Qdrant + filter |
| **`similarity_search.search`** | Внутренняя реализация с `SimilaritySearchRequest` |
| **ingestion worker** | `python -m ingestion.worker`; poll Postgres queue |
| **`run_ingest_pipeline`** | Sync ETL: raw → chunk → embed → Qdrant |
| **`ingest_jobs`** | Postgres queue / JSONL fallback; statuses `queued/running/done/failed` |
| **`SESSION_STORE`** | `memory` \| `postgres` — backend диалоговой памяти |
| **`STORAGE_PROVIDER`** | `local` \| `minio` \| `auto` — sync файлов в object storage |
| **`LMSTUDIO_EMBED_FALLBACK_ENABLED`** | Fallback embed в `lmstudio_embeddings`; **не** local chunks fallback |
| **`legacy_deprecated`** | Flag в ответе `run_orchestration`; **не** на корне `/tasks/panel` |
| **`document_retrieve`** | Legacy wrapper retrieval; pre-RAG в orchestrator |
| **`POST /chat/agent`** | Primary API: SSE, поддерживает `session_id` |

---

## Быстрые ссылки на код (top entrypoints)

| Компонент | Entry |
|-----------|-------|
| HTTP app | `app/api/main.py::create_app` |
| Agent loop | `orchestration/agent_stream.py::iter_agent_events` |
| Tool | `tools/builtin/search_kb.py::search_kb` |
| Retrieval | `retrieval/similarity_search.py::perform_similarity_search` |
| Memory factory | `memory/short_term.py::get_short_term_memory` |
| Worker | `ingestion/worker.py::run_worker_loop` |
| Pipeline | `ingestion/pipeline.py::run_ingest_pipeline` |
| Jobs | `storage/ingest_jobs.py::enqueue_ingest_job` |
