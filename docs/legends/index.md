# Легенды диаграмм

Диаграммы в этом курсе нужны не для украшения, а для архитектурного разговора. Они помогают быстро договориться, о каком контуре идет речь: пользовательский чат, агентный runtime, retrieval, ingestion, память или эксплуатация.

## Как читать диаграммы

1. Сначала определите контур: ответ пользователю, поиск источников, загрузка документов или диагностика.
2. Найдите границу ответственности: что делает агент, что делает retrieval, что делает worker.
3. Посмотрите, где появляется источник доверия: citation, job status, readiness или trace.
4. Используйте диаграмму как карту вопросов для review.

## Runbook

| Легенда | Что объясняет |
|---------|---------------|
| [Cross-area runbook](cross-area-runbook.md) | Связь пользовательского запроса, ingestion и отказов |

## По темам

| Легенда | Что объясняет |
|---------|---------------|
| [Agent runtime sequence](agent-runtime-sequence.md) | Как вопрос проходит через agent runtime |
| [Agent runtime state](agent-runtime-state.md) | Состояния обработки ответа |
| [Agent tool loop](agent-tool-loop.md) | Как агент вызывает инструменты |
| [System context](system-context.md) | Границы платформы и инфраструктуры |
| [Ingestion pipeline](ingestion-pipeline.md) | Как документы становятся знаниями |
| [Memory lifecycle](memory-lifecycle.md) | Как читается и записывается контекст сессии |
| [Worker job states](worker-job-states.md) | Жизненный цикл ingestion job |
| [search_kb sequence](search-kb-sequence.md) | Как tool получает источники |
| [Retrieval similarity search](retrieval-similarity-search.md) | Как работает смысловой поиск |
| [Executive overview](executive-overview.md) | Высокоуровневый контур для руководителей |

Полный индекс: [generated/15-diagram-legends-ru.md](../generated/15-diagram-legends-ru.md).  
Справочные файлы: [`generated/legends/`](../generated/legends/).

[← Главная](../index.md)
