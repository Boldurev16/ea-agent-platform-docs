# Легенда: Executive overview (high-level)

**Источник:** [01-executive-overview.md](../01-executive-overview.md).

## 1. Название

Simple flowchart: User → App → AgentRuntime → (optional) search_kb → Qdrant; AgentRuntime → vLLM.

## 2. Назначение

Одностраничная схема для stakeholders: RAG **on-demand** через tool, не pre-step.

## 3. Как читать

- `optional` на стрелке к `search_kb` — LLM может ответить без KB.
- Нет memory, worker, Postgres — намеренно упрощено.

## 4. Глоссарий

| Label | Кратко RU | Роль |
|-------|-----------|------|
| `App` | FastAPI platform | HTTP entry |
| `AgentRuntime` | `iter_agent_events` | Orchestration |
| `search_kb` | KB tool | On-demand RAG |
| `Qdrant` | Vector index | Evidence retrieval |
| `vLLM` | Chat model | Answer generation |

## 5. Код

`orchestration/agent_stream.py`, `tools/builtin/search_kb.py`, `llm/runtime_router.py`.

## 6. Happy path

User question about EA docs → agent calls search_kb → Qdrant → grounded answer via vLLM.

## 7. Ошибки

Диаграмма **не** показывает embed step и LM Studio — см. [agent-runtime-sequence-legend-ru.md](agent-runtime-sequence-legend-ru.md).

## 8. Документы

[01-executive-overview.md](../01-executive-overview.md), [03-system-architecture.md](../03-system-architecture.md).
