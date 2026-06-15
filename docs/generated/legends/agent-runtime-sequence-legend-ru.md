# Легенда: Agent runtime — sequence (User → SSE)

**Источник:** [03-system-architecture.md](../03-system-architecture.md) §3.1, [14-deep-dive-priority-areas.md](../14-deep-dive-priority-areas.md) §1.

## 1. Название диаграммы

Sequence: User query → `POST /chat/agent` → `iter_agent_events` → (optional) `search_kb` → SSE response.

## 2. Назначение

Показать синхронные вызовы между HTTP API, agent loop, memory, LLM, tool layer и Qdrant при одном пользовательском запросе.

## 3. Как читать эту диаграмму

- Участники сверху — runtime компоненты (не все = отдельный процесс).
- Вертикальная ось — время; стрелки вниз — порядок вызовов.
- Блок `alt action=tool search_kb` — ветка только если LLM вернул JSON `action: tool`.
- После tool loop ответ идёт клиенту как SSE; `append_turn` — после успешного завершения.

## 4. Легенда стрелок и элементов

| Элемент | Тип | Пояснение |
|---------|-----|-----------|
| `User` | Actor | Клиент (UI, curl, интегратор) |
| `FastAPI` | Boundary | `app/api/main.py` |
| `iter_agent_events` | Orchestrator | Generator agent loop |
| `Memory` | Store | `get_short_term_memory()` |
| `runtime_router` | LLM gateway | `generate_chat_completion` |
| `ToolRegistry` | Dispatcher | `tools/registry.py` |
| `EmbeddingProvider` | Embed service | Nomic → LM Studio HTTP |
| `Qdrant` | Vector DB | Cosine search |
| `->>` | Sync call | Запрос |
| `-->>` | Return | Ответ |

## 5. Глоссарий подписей

| Label | Кратко RU | Что значит | Где | Роль |
|-------|-----------|------------|-----|------|
| `POST /chat/agent` | Chat API (SSE) | Primary endpoint агента | `main.py::chat_agent` | Entry |
| `session_id` | ID сессии | Ключ server memory | `ChatAgentRequest` | Изоляция диалогов |
| `get_history` | Чтение памяти | Прошлые turns | `memory/*.py` | Context assembly |
| `generate_chat_completion` | Вызов LLM | Chat completion | `llm/runtime_router.py` | Reasoning |
| `JSON step (tool\|final)` | Шаг агента | Parsed JSON от LLM | `parse_agent_step` | Branching |
| `perform_similarity_search` | Vector search | Core retrieval | `similarity_search.py` | RAG |
| `embed_query` | Embed запроса | Query vector | `nomic_embeddings.py` | Search prep |
| `search_vectors` | Поиск в Qdrant | Top-K cosine | `qdrant_store.py` | Retrieval |
| `text/event-stream` | SSE response | Streaming events | `format_sse` | UX |
| `append_turn` | Write memory | Сохранить Q/A | `agent_stream.py:171` | Только `completed` |

## 6. Связь с кодом

| Участник | Файл / symbol |
|----------|----------------|
| API | `app/api/main.py::chat_agent` |
| Stream | `orchestration/agent_stream.py::iter_agent_events` |
| Memory | `memory/short_term.py::get_short_term_memory` |
| LLM | `llm/runtime_router.py::generate_chat_completion` |
| Tools | `tools/registry.py::ToolRegistry.call` |
| search_kb | `tools/builtin/search_kb.py` |
| Retrieval | `retrieval/similarity_search.py` |
| Embed | `embeddings/providers.py::get_embedding_provider_impl` |
| Qdrant | `vectorstore/qdrant_store.py::search_vectors` |

## 7. Happy path

1. User → `POST /chat/agent` с `query`, `session_id`.
2. `get_history` → `_build_messages` → LLM.
3. LLM → `{"action":"tool","name":"search_kb","arguments":{"query":"..."}}`.
4. `embed_query` → `search_vectors` → hits + citations в tool result.
5. LLM → `{"action":"final","answer":"...","citations":[...]}`.
6. SSE: `tool_call`, `sources`, `text`, `close`; `append_turn`.

## 8. Типичные ошибки понимания

| Ошибка | Факт (audit) |
|--------|----------------|
| «Qdrant без embed» | Между search и Qdrant всегда `embed_query` (R2) |
| «`/tasks/agent` с session» | Sync API не передаёт `session_id` (R3) |
| «sources всегда есть» | Только если tool вернул citations |
| «memory = UI history» | `dialogTurns` localStorage отдельно (R5) |

## 9. Связанные документы

- [04-agent-runtime.md](../04-agent-runtime.md)
- [06-tools-search_kb.md](../06-tools-search_kb.md)
- [11-api-and-integration-points.md](../11-api-and-integration-points.md)
- [15-documentation-audit-report.md](../15-documentation-audit-report.md) — C1–C7, R2–R5
