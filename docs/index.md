---
title: ea-agent-platform Documentation
description: Technical documentation for the corporate architect agent platform (RAG, tool loop, ingestion).
---

# ea-agent-platform — документация

Self-hosted платформа корпоративного архитектора (EA) с RAG и tool-loop агентом: индексация документов в Qdrant, on-demand поиск через `search_kb`, стриминг-чат `POST /chat/agent`, async ingest через Postgres queue.

**Статус backend:** M0–M7 ✅ (consolidation milestone complete; details in implementation repository).  
**Источник истины по коду (reverse-engineered):** каталог [`generated/`](generated/README.md), сверка — [audit report](generated/15-documentation-audit-report.md).

---

## Для кого эта документация

| Аудитория | С чего начать |
|-----------|---------------|
| **Архитектор / tech lead** | [Overview](overview/) → [Architecture](architecture/) → [Audit](audit/) |
| **Backend engineer** | [Navigation: reading paths](navigation/reading-paths.md) → [Runtime](runtime/) → [API](operations/api.md) |
| **ML / AI engineer** | [Retrieval](retrieval/) → [Runtime](runtime/) → [Memory](memory/) |
| **Ops / support** | [Operations](operations/) → [Troubleshooting map](navigation/troubleshooting-map.md) |
| **Новый участник команды** | Эта страница → [Cheat sheet](navigation/cheat-sheet.md) → [Cross-area runbook](legends/cross-area-runbook.md) |

---

## Быстрая навигация по темам

| Раздел | Содержание |
|--------|------------|
| [Overview](overview/) | Executive summary, PRD |
| [Architecture](architecture/) | System context, containers, flows |
| [Runtime](runtime/) | `iter_agent_events`, tool loop, SSE |
| [Memory](memory/) | Session memory, UI vs server |
| [Retrieval](retrieval/) | `search_kb`, `similarity_search` |
| [Ingestion](ingestion/) | Worker, pipeline, jobs |
| [Operations](operations/) | API, health, compose, risks |
| [Legends](legends/) | Легенды Mermaid-диаграмм |
| [Audit](audit/) | Documentation audit, open questions |
| [Navigation](navigation/) | Reading paths, troubleshooting, cheat sheet |
| [Generated (raw)](generated/README.md) | Полный набор reverse-engineered docs |
| [Publishing](publishing-guide.md) | GitHub Pages setup |
| [Sanitization report](publishing-sanitization-report.md) | Public export boundaries |

---

## Рекомендуемые маршруты чтения

Кратко — ниже; подробно: [navigation/reading-paths.md](navigation/reading-paths.md).

### Architect

1. [Executive overview](overview/executive-overview.md)
2. [System architecture](architecture/system-architecture.md)
3. [Data contracts](generated/10-data-contracts-and-models.md)
4. [Audit report](audit/audit-report.md)

### Backend engineer

1. [System architecture](architecture/system-architecture.md)
2. [API & integration](operations/api.md)
3. [Agent runtime](runtime/agent-runtime.md)
4. [Deep dive (6 paths)](generated/14-deep-dive-priority-areas.md)
5. [Ingestion worker](ingestion/worker.md) + [pipeline](ingestion/pipeline.md)

### ML / AI engineer

1. [similarity_search](retrieval/similarity-search.md)
2. [search_kb](retrieval/search-kb.md)
3. [Agent runtime](runtime/agent-runtime.md)
4. [Memory](memory/session-memory.md)

### Ops / support

1. [Ops & risks](operations/ops-and-risks.md)
2. [Troubleshooting map](navigation/troubleshooting-map.md)
3. [Cross-area runbook](legends/cross-area-runbook.md)
4. [Ingestion worker](ingestion/worker.md)

### New team member

1. [Cheat sheet](navigation/cheat-sheet.md)
2. [Overview](overview/) + [Architecture](architecture/)
3. [Diagram legends index](legends/)
4. [Open questions](audit/open-questions.md)

---

## Ключевые документы (прямые ссылки)

| Документ | Ссылка |
|----------|--------|
| Executive overview | [overview/executive-overview.md](overview/executive-overview.md) |
| PRD (AS-IS) | [overview/prd.md](overview/prd.md) |
| System architecture | [architecture/system-architecture.md](architecture/system-architecture.md) |
| Agent runtime | [runtime/agent-runtime.md](runtime/agent-runtime.md) |
| Memory | [memory/session-memory.md](memory/session-memory.md) |
| Tool `search_kb` | [retrieval/search-kb.md](retrieval/search-kb.md) |
| `similarity_search` | [retrieval/similarity-search.md](retrieval/similarity-search.md) |
| Ingestion worker | [ingestion/worker.md](ingestion/worker.md) |
| Ingestion pipeline | [ingestion/pipeline.md](ingestion/pipeline.md) |
| Diagram legends | [legends/](legends/) |
| Audit report | [audit/audit-report.md](audit/audit-report.md) |

---

## Scope of this public repository

This site publishes **sanitized architecture and operations documentation**. Internal dated runbooks, ADR drafts, eval artifacts, and planning documents remain in the private implementation repository.

See [publishing-sanitization-report.md](publishing-sanitization-report.md).

---

## Публикация на GitHub Pages

1. Repository **Settings → Pages**.
2. Source: branch (например `main`) → folder **`/docs`**.
3. Entry point: этот файл **`docs/index.md`**.
4. После deploy: `https://<org>.github.io/<repo>/` (или custom domain).

Подробнее: [navigation/github-pages.md](navigation/github-pages.md).

---

## Легенда достоверности (generated docs)

- **Confirmed by code** — подтверждено кодом/тестами
- **Inferred** — логический вывод из структуры
- **Needs verification** — требует ручной проверки (см. [audit](audit/))
