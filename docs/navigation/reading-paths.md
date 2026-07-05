# Рекомендуемые маршруты чтения

Эта страница помогает выбрать глубину погружения. Агентная платформа затрагивает сразу несколько контуров: бизнес-ценность, корпоративную архитектуру, AI/runtime, данные, эксплуатацию и безопасность. Поэтому читать все подряд не нужно: начните с роли.

## C-level / руководитель архитектурной функции

**Цель:** понять, зачем компании нужен агентный контур управления архитектурой и какие решения он делает более прозрачными.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [Executive overview](../overview/executive-overview.md) | Бизнес-смысл: скорость, единая база знаний, трассируемость |
| 2 | [PRD](../overview/prd.md) | Границы продукта и пользовательские сценарии |
| 3 | [System architecture](../architecture/system-architecture.md) | Как компоненты складываются в управляемый контур |
| 4 | [Open questions](../audit/open-questions.md) | Какие решения требуют управленческого выбора |

**Ключевая мысль:** платформа не заменяет архитектурную функцию; она усиливает ее, делая знания доступными, ответы воспроизводимыми, а источники проверяемыми.

## Enterprise architect / главный архитектор

**Цель:** увидеть, как система поддерживает архитектурное управление: от базы знаний до рекомендаций с источниками.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [Cheat sheet](cheat-sheet.md) | Карта платформы и терминов |
| 2 | [System architecture](../architecture/system-architecture.md) | Слои: интерфейс, агент, tools, RAG, инфраструктура |
| 3 | [search_kb](../retrieval/search-kb.md) | Как агент обращается к знаниям |
| 4 | [similarity_search](../retrieval/similarity-search.md) | Как выбираются релевантные фрагменты |
| 5 | [Audit report](../audit/audit-report.md) | Что подтверждено и где остаются риски |

**Ключевая мысль:** RAG нужен не ради “поиска документов”, а ради управляемой аргументации: ответ должен иметь источник, область применимости и проверяемый след.

## Product owner / владелец продукта

**Цель:** понять текущий MVP, разницу с полноценным продуктовым интерфейсом и приоритеты развития.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [PRD](../overview/prd.md) | Что входит в текущий продуктовый контур |
| 2 | [Executive overview](../overview/executive-overview.md) | Пользовательская ценность |
| 3 | [Operations risks](../operations/ops-and-risks.md) | Ограничения внедрения |
| 4 | [Open questions](../audit/open-questions.md) | Backlog решений для продуктового развития |

**Ключевая мысль:** текущая платформа уже закрывает backend RAG+agent, но требует продуктовой оболочки: workspaces, загрузка документов, роли, настройки.

## Инженерная команда

**Цель:** понять execution paths: как запрос превращается в ответ, а документ — в searchable knowledge.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [System architecture](../architecture/system-architecture.md) | Контейнерная и компонентная карта |
| 2 | [API & integration](../operations/api.md) | Основные endpoint и их назначение |
| 3 | [Agent runtime](../runtime/agent-runtime.md) | Tool loop, события SSE, финальный ответ |
| 4 | [Ingestion pipeline](../ingestion/pipeline.md) | Parse → chunk → embed → index |
| 5 | [Ingestion worker](../ingestion/worker.md) | Очередь заданий и статусы обработки |
| 6 | [Deep dive](../generated/14-deep-dive-priority-areas.md) | Справочные пути для глубокого разбора |

## ML / AI инженер

**Цель:** понять качество ответа: embeddings, retrieval, tool behavior, память и ограничения.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [Agent runtime](../runtime/agent-runtime.md) | Как LLM принимает решение о tool call |
| 2 | [search_kb](../retrieval/search-kb.md) | Контракт инструмента поиска |
| 3 | [similarity_search](../retrieval/similarity-search.md) | Семантический поиск и фильтрация |
| 4 | [Memory](../memory/session-memory.md) | Контекст сессии не равен корпоративной памяти |
| 5 | [Retrieval legend](../legends/retrieval-similarity-search.md) | Диаграмма поиска с пояснениями |

## Эксплуатация / support

**Цель:** быстро определить, какой контур деградировал: приложение, модель, embeddings, Qdrant, Postgres, ingestion.

| Шаг | Документ | Что вынести |
|-----|----------|-------------|
| 1 | [Ops & risks](../operations/ops-and-risks.md) | Health, риски и эксплуатационные сценарии |
| 2 | [Troubleshooting map](troubleshooting-map.md) | Куда смотреть при типовых симптомах |
| 3 | [API health endpoints](../operations/api.md) | Разница `/health/live` и `/health/ready` |
| 4 | [Ingestion worker](../ingestion/worker.md) | Очередь, `running`, `done`, `failed` |
| 5 | [Cross-area runbook](../legends/cross-area-runbook.md) | Связь запроса, ingestion и поиска |

## Новый участник команды

| День | Задача | Документы |
|------|--------|-----------|
| День 1 | Понять назначение и карту системы | [index](../index.md) → [cheat sheet](cheat-sheet.md) → [executive overview](../overview/executive-overview.md) |
| День 2 | Разобрать agent runtime и RAG | [agent runtime](../runtime/agent-runtime.md) → [search_kb](../retrieval/search-kb.md) → [similarity_search](../retrieval/similarity-search.md) |
| День 3 | Понять эксплуатацию | [API](../operations/api.md) → [worker](../ingestion/worker.md) → [troubleshooting](troubleshooting-map.md) |

## Связанные страницы

- [Глоссарий](glossary.md)
- [Архитектурная шпаргалка](cheat-sheet.md)
- [Troubleshooting map](troubleshooting-map.md)
- [Generated reference](../generated/README.md)
