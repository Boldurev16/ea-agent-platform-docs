# 15 — Индекс легенд Mermaid-диаграмм

Диаграммы в курсе показывают не только технические вызовы, но и архитектурные контуры: где возникает решение, где появляется источник, где фиксируется состояние и где возможна деградация.

## Как читать диаграммы

| Тип | Как читать |
|-----|------------|
| `flowchart` | Блоки — компоненты или этапы; стрелки — направление данных или ответственности. |
| `sequenceDiagram` | Участники — системы или роли; стрелки — вызовы и ответы. |
| `stateDiagram-v2` | Узлы — состояния; стрелки — условия перехода. |

## Главные диаграммы курса

| Тема | Документ | Что показывает |
|------|----------|----------------|
| Executive overview | [01-executive-overview.md](01-executive-overview.md) | Как пользователь получает проверяемый ответ |
| System architecture | [03-system-architecture.md](03-system-architecture.md) | Все контуры платформы |
| Agent runtime | [04-agent-runtime.md](04-agent-runtime.md) | Жизненный цикл ответа |
| Memory | [05-memory.md](05-memory.md) | Как сессия попадает в контекст |
| search_kb | [06-tools-search_kb.md](06-tools-search_kb.md) | Как агент обращается к знаниям |
| similarity_search | [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md) | Как выбираются фрагменты |
| Ingestion worker | [08-ingestion-worker.md](08-ingestion-worker.md) | Статусы обработки документов |
| Ingestion pipeline | [09-ingestion-pipeline.md](09-ingestion-pipeline.md) | Документ → база знаний |
| Deep dive | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) | Связь всех критических контуров |

## Словарь обозначений

| Обозначение | Значение |
|-------------|----------|
| Agent runtime | Контур рассуждения и выбора действий. |
| Tool | Разрешенное действие агента. |
| Retrieval | Семантический поиск по базе знаний. |
| Citation | Источник, который подтверждает ответ. |
| Worker | Исполнитель ingestion-задач. |
| Readiness | Готовность всего контура. |
| Memory | Контекст диалога, не источник истины. |

## Файлы легенд

- [agent-runtime-sequence-legend-ru.md](legends/agent-runtime-sequence-legend-ru.md)
- [agent-runtime-state-legend-ru.md](legends/agent-runtime-state-legend-ru.md)
- [agent-tool-loop-legend-ru.md](legends/agent-tool-loop-legend-ru.md)
- [system-context-legend-ru.md](legends/system-context-legend-ru.md)
- [ingestion-pipeline-legend-ru.md](legends/ingestion-pipeline-legend-ru.md)
- [memory-lifecycle-legend-ru.md](legends/memory-lifecycle-legend-ru.md)
- [worker-job-states-legend-ru.md](legends/worker-job-states-legend-ru.md)
- [search-kb-sequence-legend-ru.md](legends/search-kb-sequence-legend-ru.md)
- [retrieval-similarity-search-legend-ru.md](legends/retrieval-similarity-search-legend-ru.md)
- [cross-area-map-legend-ru.md](legends/cross-area-map-legend-ru.md)
- [executive-overview-legend-ru.md](legends/executive-overview-legend-ru.md)

## Практика использования

На архитектурном обсуждении начинайте с диаграммы system architecture, затем переходите к конкретному контуру: runtime, retrieval, ingestion или operations. Это помогает не смешивать пользовательский сценарий, внутренний tool loop и эксплуатационные зависимости.
