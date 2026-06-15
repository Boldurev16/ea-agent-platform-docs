# Генерированная документация ea-agent-platform

> **Читателям:** начните с [документационного landing page](../index.md) (GitHub Pages entry).  
> Этот каталог — полный reverse-engineered source.

Дата генерации: `2026-06-07`.  
Метод: reverse-engineering из актуального кода репозитория `ea-agent-platform` (implementation repo — not published here).

## Карта документов

| # | Документ | Аудитория |
|---|----------|-----------|
| 01 | [01-executive-overview.md](01-executive-overview.md) | Stakeholders, tech leads |
| 02 | [02-prd.md](02-prd.md) | Product, архитекторы |
| 03 | [03-system-architecture.md](03-system-architecture.md) | Architects, backend |
| 04 | [04-agent-runtime.md](04-agent-runtime.md) | AI/backend engineers |
| 05 | [05-memory.md](05-memory.md) | Backend, AI |
| 06 | [06-tools-search_kb.md](06-tools-search_kb.md) | Backend, AI |
| 07 | [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md) | Backend, ML |
| 08 | [08-ingestion-worker.md](08-ingestion-worker.md) | Backend, ops |
| 09 | [09-ingestion-pipeline.md](09-ingestion-pipeline.md) | Backend, data |
| 10 | [10-data-contracts-and-models.md](10-data-contracts-and-models.md) | All engineers |
| 11 | [11-api-and-integration-points.md](11-api-and-integration-points.md) | Integrators, frontend |
| 12 | [12-ops-observability-and-risks.md](12-ops-observability-and-risks.md) | SRE, ops |
| 13 | [13-open-questions.md](13-open-questions.md) | Все — backlog валидации |
| 14 | [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) | **Deep dive:** runtime, memory, tools, retrieval, worker, pipeline |
| 15 | [15-documentation-audit-report.md](15-documentation-audit-report.md) | **Audit:** confirmed / corrected / gaps |

## Рекомендуемый порядок чтения

### Executives / stakeholders
1. `01-executive-overview.md`
2. `02-prd.md` (разделы In-scope / Out-of-scope)
3. `13-open-questions.md` (риски и gap)

### Architects
1. `01-executive-overview.md`
2. `03-system-architecture.md`
3. `04-agent-runtime.md`
4. `10-data-contracts-and-models.md`
5. `13-open-questions.md`

### Backend engineers (onboarding)
1. [14-deep-dive-priority-areas.md](14-deep-dive-priority-areas.md) — **start here** for execution paths
2. `03-system-architecture.md`
2. `11-api-and-integration-points.md`
3. `04-agent-runtime.md`
4. `09-ingestion-pipeline.md`
5. `08-ingestion-worker.md`

### ML / AI engineers
1. `04-agent-runtime.md`
2. `07-retrieval-similarity_search.md`
3. `06-tools-search_kb.md`
4. `05-memory.md`
5. `10-data-contracts-and-models.md`

### Operations / support
1. `12-ops-observability-and-risks.md`
2. `11-api-and-integration-points.md` (health endpoints)
3. `08-ingestion-worker.md`
4. `09-ingestion-pipeline.md`

## Статус реализации (snapshot)

| Фаза | Статус |
|------|--------|
| M0–M7 backend | ✅ COMPLETE |
| Phase 2 P1–P4 product | ⬜ PROPOSED — see private implementation repository planning docs |

## Легенда достоверности

В документах используются маркеры:

- **Confirmed by code** — прямое подтверждение в коде/тестах
- **Inferred** — логическое следствие из структуры, не явный runtime path
- **Needs verification** — требует ручной проверки или уточнения с командой
