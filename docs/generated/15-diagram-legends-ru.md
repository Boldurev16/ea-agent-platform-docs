# Индекс легенд Mermaid-диаграмм

Русскоязычные легенды для всех диаграмм в `docs/generated/`.  
Шпаргалка навигации: [14-docs-cheat-sheet-ru.md](14-docs-cheat-sheet-ru.md).

---

## Как найти диаграмму в документации

1. В файле `docs/generated/*.md` ищите блок ` ```mermaid `.
2. Заголовок секции над блоком — **имя диаграммы** (например «### 3.1 User query → agent → response»).
3. Соответствующая легенда — в `docs/generated/legends/` (ссылка в таблице ниже).
4. Диаграммы **не вынесены** в отдельные файлы — они встроены в topic docs; легенды объясняют содержимое без копирования SVG.

**Именование легенд:** `{тема}-legend-ru.md` — одна легенда может покрывать несколько похожих диаграмм в разных файлах.

---

## Как читать диаграммы в этом проекте

### Типы диаграмм

| Тип Mermaid | Как читать |
|-------------|------------|
| **flowchart** (`flowchart TB/LR/TD`) | Блоки = компоненты или шаги; стрелка = порядок или вызов. `subgraph` = логическая граница (акторы, внешние сервисы). |
| **sequenceDiagram** | Участники (lifelines) сверху; стрелка `->>` синхронный вызов; `-->>` ответ; `alt` ветвление. |
| **stateDiagram-v2** | Состояния и переходы; подпись на стрелке = условие перехода. |

### Стрелки (общая нотация)

| Символ | Meaning |
|--------|---------|
| `-->` / `->>` | Направление потока данных или control flow |
| Пунктир в Mermaid | В этом проекте редко; если есть — обычно optional path |
| `alt` / `opt` | Условная ветка (например `action=tool`) |

### Условные обозначения

- **Прямой путь (solid)** — основной happy path по коду.
- **Optional** в подписи — шаг может быть пропущен (например MinIO sync, tool call).
- **External** — процесс вне `ea-app` container (vLLM, LM Studio на host).

### Соответствие audit report

Легенды **не** воспроизводят устаревшие утверждения из audit §2 (Corrected). При сомнении — [15-documentation-audit-report.md](15-documentation-audit-report.md).

---

## Реестр диаграмм

| # | Имя / секция | Файл источника | Тип | Легенда |
|---|--------------|----------------|-----|---------|
| 1 | System context | [03-system-architecture.md](03-system-architecture.md) §1 | flowchart | [legends/system-context-legend-ru.md](legends/system-context-legend-ru.md) |
| 2 | User query → agent → response | [03-system-architecture.md](03-system-architecture.md) §3.1 | sequence | [legends/agent-runtime-sequence-legend-ru.md](legends/agent-runtime-sequence-legend-ru.md) |
| 3 | Agent request lifecycle (state) | [04-agent-runtime.md](04-agent-runtime.md) | stateDiagram | [legends/agent-runtime-state-legend-ru.md](legends/agent-runtime-state-legend-ru.md) |
| 4 | Tool invocation lifecycle | [04-agent-runtime.md](04-agent-runtime.md) | sequence | [legends/agent-tool-loop-legend-ru.md](legends/agent-tool-loop-legend-ru.md) |
| 5 | Executive overview (high-level) | [01-executive-overview.md](01-executive-overview.md) | flowchart | [legends/executive-overview-legend-ru.md](legends/executive-overview-legend-ru.md) |
| 6 | Ingestion pipeline (architecture) | [03-system-architecture.md](03-system-architecture.md) §3.2 | flowchart | [legends/ingestion-pipeline-legend-ru.md](legends/ingestion-pipeline-legend-ru.md) |
| 7 | Ingestion pipeline (numbered stages) | [09-ingestion-pipeline.md](09-ingestion-pipeline.md) | flowchart | [legends/ingestion-pipeline-legend-ru.md](legends/ingestion-pipeline-legend-ru.md) |
| 8 | Memory lifecycle (state, arch) | [03-system-architecture.md](03-system-architecture.md) §3.3 | stateDiagram | [legends/memory-lifecycle-legend-ru.md](legends/memory-lifecycle-legend-ru.md) |
| 9 | Memory lifecycle (flow, detail) | [05-memory.md](05-memory.md) | flowchart | [legends/memory-lifecycle-legend-ru.md](legends/memory-lifecycle-legend-ru.md) |
| 10 | Worker job states | [08-ingestion-worker.md](08-ingestion-worker.md) | stateDiagram | [legends/worker-job-states-legend-ru.md](legends/worker-job-states-legend-ru.md) |
| 11 | Worker enqueue → poll sequence | [08-ingestion-worker.md](08-ingestion-worker.md) | sequence | [legends/worker-job-states-legend-ru.md](legends/worker-job-states-legend-ru.md) |
| 12 | Retrieval internal flow | [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md) | flowchart | [legends/retrieval-similarity-search-legend-ru.md](legends/retrieval-similarity-search-legend-ru.md) |
| 13 | `similarity_search` sequence | [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md) | sequence | [legends/retrieval-similarity-search-legend-ru.md](legends/retrieval-similarity-search-legend-ru.md) |
| 14 | `search_kb` runtime call chain | [06-tools-search_kb.md](06-tools-search_kb.md) | sequence | [legends/search-kb-sequence-legend-ru.md](legends/search-kb-sequence-legend-ru.md) |
| 15 | Cross-area interaction map | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) | flowchart | [legends/cross-area-map-legend-ru.md](legends/cross-area-map-legend-ru.md) |
| 16 | Agent area sequence (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §1 | sequence | [legends/agent-runtime-sequence-legend-ru.md](legends/agent-runtime-sequence-legend-ru.md) |
| 17 | Memory state (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §2 | stateDiagram | [legends/memory-lifecycle-legend-ru.md](legends/memory-lifecycle-legend-ru.md) |
| 18 | search_kb chain (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §3 | sequence | [legends/search-kb-sequence-legend-ru.md](legends/search-kb-sequence-legend-ru.md) |
| 19 | Retrieval flow (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §4 | flowchart | [legends/retrieval-similarity-search-legend-ru.md](legends/retrieval-similarity-search-legend-ru.md) |
| 20 | Worker lifecycle (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §5 | stateDiagram | [legends/worker-job-states-legend-ru.md](legends/worker-job-states-legend-ru.md) |
| 21 | Pipeline E2E (deep dive) | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) §6 | flowchart | [legends/ingestion-pipeline-legend-ru.md](legends/ingestion-pipeline-legend-ru.md) |

---

## Обязательные метки (глобальный словарь)

Сводная таблица частых подписей на диаграммах — детали в легендах по темам.

| Label (original) | Коротко RU | Роль в E2E |
|------------------|------------|------------|
| `Op` / Operator | Оператор KB | Запускает ingest, кладёт файлы в `data/raw` / inbox |
| `POST /ingestion/run` | API async ingest | Enqueue job → `ingest_jobs` |
| `similarity_search` | Модуль vector retrieval | `retrieval/similarity_search.py` |
| `search_kb` | Tool агента | On-demand поиск в KB |
| `worker` | `ingestion/worker` | Consumer очереди ingest |
| `pipeline` / `run_ingest_pipeline` | Sync ETL | Parse → chunk → embed → Qdrant |
| `runtime` / `iter_agent_events` | Agent runtime | Tool loop + SSE |
| `memory read` / `get_history` | Чтение session turns | Перед LLM call |
| `memory write` / `append_turn` | Запись turn | После `status=completed` |
| `Qdrant` / `vector store` | Векторный индекс | Cosine search, upsert |
| `chunk` / `chunk(220)` | Фрагмент текста | max 220 chars при ingest |
| `embed_query` | Векторизация запроса | Search path |
| `embed_texts` | Batch векторизация | Ingest path |
| `queue` / `ingest_jobs` | Postgres job queue | Async ingest |
| `session_id` | Ключ server memory | **Не** `dialogTurns` |
| `legacy_deprecated` | Flag deprecated API | В `run_orchestration` response |
| `document_retrieve` | Legacy retrieval wrapper | Pre-RAG, не agent tool path |
| `LMSTUDIO_EMBED_FALLBACK_ENABLED` | Env fallback embed | `lmstudio_embeddings.py` |
| `SESSION_STORE` | Backend памяти | `memory` \| `postgres` |
| `STORAGE_PROVIDER` | Object storage mode | MinIO sync optional |

---

## Файлы легенд

| Файл | Покрывает |
|------|-----------|
| [legends/agent-runtime-sequence-legend-ru.md](legends/agent-runtime-sequence-legend-ru.md) | Agent HTTP → SSE sequence |
| [legends/agent-runtime-state-legend-ru.md](legends/agent-runtime-state-legend-ru.md) | Agent state machine |
| [legends/agent-tool-loop-legend-ru.md](legends/agent-tool-loop-legend-ru.md) | ToolRegistry + dedupe |
| [legends/system-context-legend-ru.md](legends/system-context-legend-ru.md) | C4 context |
| [legends/ingestion-pipeline-legend-ru.md](legends/ingestion-pipeline-legend-ru.md) | Pipeline flowcharts |
| [legends/memory-lifecycle-legend-ru.md](legends/memory-lifecycle-legend-ru.md) | Memory read/write |
| [legends/worker-job-states-legend-ru.md](legends/worker-job-states-legend-ru.md) | Worker + jobs |
| [legends/search-kb-sequence-legend-ru.md](legends/search-kb-sequence-legend-ru.md) | search_kb chains |
| [legends/retrieval-similarity-search-legend-ru.md](legends/retrieval-similarity-search-legend-ru.md) | similarity_search |
| [legends/cross-area-map-legend-ru.md](legends/cross-area-map-legend-ru.md) | End-to-end map |
| [legends/executive-overview-legend-ru.md](legends/executive-overview-legend-ru.md) | Simple agent+RAG |
