# Легенда: Agent runtime — state machine

**Источник:** [04-agent-runtime.md](../04-agent-runtime.md) Request/Session Lifecycle.

## 1. Название

State diagram: жизненный цикл одного запроса в `iter_agent_events`.

## 2. Назначение

Показать все терминальные состояния (`completed`, `error`, `max_rounds`) и ветки (tool, dedupe, parse fallback).

## 3. Как читать

- Каждый узел — фаза обработки одного HTTP/SSE запроса.
- Переходы с подписью — условие из кода.
- `Close` — финальный SSE event `close`, не HTTP disconnect до конца stream.

## 4. Легенда элементов

| Узел | Смысл |
|------|-------|
| `Started` | `yield status started` |
| `LLMCall` | `generate(messages)` |
| `ParseStep` | `parse_agent_step` |
| `ToolExec` | `registry.call` |
| `DedupeCheck` | `ToolCallDeduper.should_block` |
| `StreamText` | `_chunk_text(answer)` → SSE `text` |
| `PersistMemory` | `append_turn` |
| `Close` | SSE `close` |

## 5. Глоссарий

| Label | Кратко RU | Что значит | Где | Роль |
|-------|-----------|------------|-----|------|
| `AGENT_MAX_TOOL_CALLS` | Лимит tools | Default 5 | `runtime_settings.py` | Stop condition |
| `ToolCallDeduper` | Анти-дубликат | Same tool+args blocked | `tools/dedupe.py` | Guard |
| `parse_fallback` | Fallback parse | Сырой текст LLM как answer | `agent_stream.py:89-93` | Resilience |
| `max_rounds` | Лимит исчерпан | `status=max_rounds` | `agent_stream.py:111-115` | Termination |
| `status=error` | LLM down | Double `generate` None | `agent_stream.py:76-81` | Failure |
| `introspect` | Debug SSE | trace + tool_calls count | `agent_stream.py:168-169` | Observability |

## 6. Связь с кодом

`orchestration/agent_stream.py::iter_agent_events` — единственный orchestrator state machine.

## 7. Happy path

`Started` → `LLMCall` → `ParseStep` → `ToolExec` (×1) → `LLMCall` → `Final` → `StreamText` → `Introspect` → `PersistMemory` → `Close`.

## 8. Типичные ошибки

- **PersistMemory не всегда:** при `error`, `max_rounds`, parse fallback с non-completed — write может не происходить; write gate — `status==completed`.
- **Dedupe увеличивает `tool_calls`:** blocked call тоже считается в лимите.

## 9. Документы

[04-agent-runtime.md](../04-agent-runtime.md), [14-deep-dive-priority-areas.md](../14-deep-dive-priority-areas.md) §1, audit C8, C9.
