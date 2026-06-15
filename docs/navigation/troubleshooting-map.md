# Troubleshooting map — куда смотреть, если…

Incident-oriented навигация. Факты согласованы с [audit report](../generated/15-documentation-audit-report.md).  
Runbook по двум потокам: [Cross-area runbook](../legends/cross-area-runbook.md).

---

## `search_kb` возвращает не те результаты / пустые hits

| | |
|---|---|
| **Симптомы** | Неверные citations, пустой tool result, низкий score |
| **Документы** | [search_kb](../retrieval/search-kb.md), [similarity_search](../retrieval/similarity-search.md), [Ingestion pipeline](../ingestion/pipeline.md) |
| **Код** | `tools/builtin/search_kb.py`, `retrieval/similarity_search.py` |
| **Проверки** | Ingest выполнен? `data/ingestion_report.json`; Qdrant up — `GET /health/ready`; threshold `0.25`, `top_k` |
| **Audit** | C2, C3, R10 — agent path **без** local `chunks.json` fallback |

---

## `similarity_search` — мало или слишком много chunks

| | |
|---|---|
| **Симптомы** | `filtered_hits=0` в debug; слишком широкий контекст |
| **Документы** | [similarity_search](../retrieval/similarity-search.md), [Data contracts](../generated/10-data-contracts-and-models.md) |
| **Проверки** | `similarity_threshold`, `top_k`, `CORPUS_ID_DEFAULT`; chunk size 220 при ingest |
| **Audit** | R13 — `filter_identifiers` исключает listed; gap — `RetrievalRequest.filters` не применяются |

---

## Agent runtime: много tool calls, нет `sources`, падения

| | |
|---|---|
| **Симптомы** | `status=max_rounds`, `status=error`, нет SSE `sources`, `parse_fallback` |
| **Документы** | [Agent runtime](../runtime/agent-runtime.md), [API](../operations/api.md), [Agent state legend](../legends/agent-runtime-state.md) |
| **Проверки** | `GET /health/llm`; SSE `introspect`; `AGENT_MAX_TOOL_CALLS` (default 5); dedupe в trace |
| **Audit** | C7, C8; R3 — `POST /tasks/agent` **без** `session_id` |

---

## Memory выглядит непоследовательной

| | |
|---|---|
| **Симптомы** | Агент «не помнит» прошлый turn; разные ответы при том же UI |
| **Документы** | [Session memory](../memory/session-memory.md), [API](../operations/api.md) |
| **Проверки** | `session_id` в `POST /chat/agent`; `SESSION_STORE=postgres` vs memory; не путать с UI |
| **Audit** | R5 — server memory ≠ `dialogTurns` localStorage |

---

## UI history ≠ server memory

| | |
|---|---|
| **Симптомы** | Панель истории полная, агент без контекста; или наоборот |
| **Документы** | [Session memory](../memory/session-memory.md), [Troubleshooting: clear history](#clear-history-в-ui-не-очищает-серверную-память) |
| **Код** | `app/api/main.py` — `dialogTurns`, `ea_agent_session_id_v1` |
| **Audit** | R4, R5; gap — нет HTTP API для `memory.clear()` |

---

## Clear history в UI не очищает серверную память

| | |
|---|---|
| **Ожидание vs факт** | Кнопка «Очистить историю» → только `localStorage` (`clearHistory()`) |
| **Документы** | [Session memory](../memory/session-memory.md), [API](../operations/api.md) |
| **Workaround** | Новый `session_id` в `POST /chat/agent` |
| **Audit** | R4 — confirmed behavior |

---

## Ingestion сломан / job `failed`

| | |
|---|---|
| **Симптомы** | `POST /ingestion/run` → `failed`; пустой Qdrant |
| **Документы** | [Pipeline](../ingestion/pipeline.md), [Worker](../ingestion/worker.md), [Ops](../operations/ops-and-risks.md) |
| **Проверки** | `GET /ingestion/jobs/{job_id}`; `data/raw/` есть parseable files; LM Studio embed; `ingestion_skipped.json` |
| **Audit** | R14 — parse из локального `data/raw/`, не MinIO read; U4 batch embed |

---

## Worker jobs застревают в `running`

| | |
|---|---|
| **Симптомы** | Job долго `running`; worker restart |
| **Документы** | [Worker](../ingestion/worker.md), [Cross-area runbook](../legends/cross-area-runbook.md) |
| **Проверки** | Postgres `ingest_jobs`; логи `ea-worker`; manual status fix — **нет auto recovery в коде** |
| **Audit** | U2, U3, R16 — confirmed gap stale recovery |

---

## LM Studio / embedding issues

| | |
|---|---|
| **Симптомы** | Ingest fail; `SimilaritySearchError` в agent; «успешный» ingest с плохим search |
| **Документы** | [similarity_search](../retrieval/similarity-search.md), [Ops](../operations/ops-and-risks.md), [Pipeline](../ingestion/pipeline.md) |
| **Проверки** | `GET /health/ready` → embeddings; `LMSTUDIO_EMBED_FALLBACK_ENABLED`; compose env |
| **Audit** | R10, U5 — fallback в `lmstudio_embeddings`, **не** в `similarity_search` / agent path |

---

## Readiness `degraded`, но API отвечает

| | |
|---|---|
| **Симптомы** | HTTP 200, `status: degraded` в `/health/ready` |
| **Документы** | [Ops](../operations/ops-and-risks.md), [API](../operations/api.md) |
| **Audit** | C22 — by design; см. поле `blocking` |

---

## Legacy API vs primary agent (`/tasks/panel`, `/tasks/orchestrate`)

| | |
|---|---|
| **Симптомы** | Другие hits/citations vs `/chat/agent` |
| **Документы** | [API](../operations/api.md), [PRD](../overview/prd.md) |
| **Факт** | Legacy — pre-RAG `document_retrieve`; `legacy_deprecated` в nested `orchestrator`, не panel root |
| **Audit** | C19, R1, U10 |

---

## Periodic worker / inbox files не попадают в raw

| | |
|---|---|
| **Документы** | [Pipeline](../ingestion/pipeline.md) |
| **Факт** | Только `.pptx`, `.pdf`, `.csv`, `.xlsx`, `.xls`, `.docx`; mtime-based copy |
| **Audit** | R6, U1 |

---

## Быстрые health checks

| Endpoint | Назначение |
|----------|------------|
| `GET /health/live` | Liveness |
| `GET /health/ready` | Qdrant, embeddings, postgres |
| `GET /health/llm` | Chat runtime + embeddings snapshot |

Smoke: см. [Ops & risks](../operations/ops-and-risks.md).

---

## Связанные страницы

- [Reading paths](reading-paths.md)
- [Cheat sheet](cheat-sheet.md)
- [Open questions](../audit/open-questions.md)
