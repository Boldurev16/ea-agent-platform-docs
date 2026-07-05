# Легенда: Agent tool loop (ToolRegistry + dedupe)

**Источник:** [04-agent-runtime.md](../04-agent-runtime.md) Tool Invocation Lifecycle.

## 1. Название

Sequence: `agent_stream` → `parse_agent_step` → `ToolCallDeduper` → `ToolRegistry` → `search_kb`.

## 2. Назначение

Детализировать tool path **внутри** agent loop (без HTTP и без full retrieval stack).

## 3. Как читать

- Участник `parse_agent_step` — парсер, не отдельный сервис.
- `alt not blocked` — tool выполняется только если dedupe разрешил.

## 4. Стрелки

| Стрелка | Смысл |
|---------|-------|
| AS→P | Текст ответа LLM → структурированный JSON-шаг |
| AS→D | Проверка fingerprint tool+args |
| AS→R→T | Dispatch tool handler |

## 5. Глоссарий

| Label | Кратко RU | Где | Роль |
|-------|-----------|-----|------|
| `action=tool` | Вызов инструмента | LLM JSON | Branch |
| `name` | Имя tool | e.g. `search_kb` | Registry key |
| `arguments` | Параметры tool | dict from JSON | Handler kwargs |
| `should_block` | Dedupe check | `ToolCallDeduper` | Prevent repeat |
| `tool_result` | JSON dict | Serialized to messages | LLM context |
| `hits, citations, debug` | Output search_kb | Tool return | Citations SSE |

## 6. Код

- `parsers/agent_response.py::parse_agent_step`
- `tools/dedupe.py::ToolCallDeduper`
- `tools/registry.py::ToolRegistry`
- `orchestration/agent_runtime.py::_default_registry` — только `search_kb`

## 7. Happy path

Parse tool step → dedupe pass → `call("search_kb")` → tool_result в messages → следующий LLM call.

## 8. Ошибки понимания

- Unknown tool → `{error}` в result, loop **продолжается** (не HTTP 500).
- Полный retrieval chain — см. [search-kb-sequence-legend-ru.md](search-kb-sequence-legend-ru.md).

## 9. Документы

[04-agent-runtime.md](../04-agent-runtime.md), [06-tools-search_kb.md](../06-tools-search_kb.md).
