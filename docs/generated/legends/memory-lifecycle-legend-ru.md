# Легенда: Memory lifecycle

**Источник:** [03-system-architecture.md](../03-system-architecture.md) §3.3, [05-memory.md](../05-memory.md), [14-deep-dive](../14-deep-dive-priority-areas.md) §2.

## 1. Название

Memory read/write: `session_id` → history → prompt → conditional write.

## 2. Назначение

Показать **server-side** session memory (не UI `dialogTurns`).

## 3. Как читать

- State diagram (03): упрощённый write gate `status=completed`.
- Flowchart (05): выбор backend `SESSION_STORE`.
- Write **не** происходит при error/max_rounds.

## 4. Элементы

| Элемент | Смысл |
|---------|-------|
| `get_short_term_memory()` | Factory |
| `ShortTermMemory` | In-process dict |
| `PostgresShortTermMemory` | Table `chat_turns` |
| `_build_messages` | Inject turns into LLM messages |
| `append_turn` | Write Q/A pair |
| `Trim max 10` | Sliding window |

## 5. Глоссарий

| Label | Кратко RU | Где | Роль |
|-------|-----------|-----|------|
| `session_id` | Ключ сессии | `ChatAgentRequest` | Memory partition |
| `SESSION_STORE` | `memory` \| `postgres` | env | Backend selection |
| `memory read` | `get_history` | Start of request | Context |
| `memory write` | `append_turn` | End if completed | Persistence |
| `dialogTurns` | UI localStorage | **Не на диаграмме** | Display only (R5) |
| `context` | `messages[]` | Agent loop | Working context |
| `state` | Per-request vars | trace, citations | Not persisted |

## 6. Код

- `memory/short_term.py`, `memory/postgres_store.py`
- `orchestration/agent_stream.py::_build_messages`, lines 171-172

## 7. Happy path

`POST /chat/agent` + `session_id` → read turns → agent loop → `completed` → append → trim 10.

## 8. Ошибки понимания

| Ошибка | Факт |
|--------|------|
| Clear UI = clear server | `clearHistory()` только localStorage (R4) |
| `POST /tasks/agent` multi-session | Всегда `default` (R3) |
| Tool traces в memory | Сохраняется только final answer |

**Gap:** нет HTTP API для `memory.clear()`.

## 9. Документы

[05-memory.md](../05-memory.md), [11-api](../11-api-and-integration-points.md), audit C9–C11, R3–R5.
