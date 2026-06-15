# Рекомендуемые маршруты чтения

Порядок чтения по ролям. Технические идентификаторы — в оригинале.  
Сверка с кодом: [audit report](../generated/15-documentation-audit-report.md).

---

## Архитектор / principal engineer

**Цель:** границы системы, контракты, риски, что подтверждено audit.

| # | Документ | Зачем |
|---|----------|-------|
| 1 | [Executive overview](../overview/executive-overview.md) | Scope, in/out, stakeholders |
| 2 | [PRD](../overview/prd.md) | Journeys, NFR из кода |
| 3 | [System architecture](../architecture/system-architecture.md) | Context, compose, Mermaid flows |
| 4 | [Data contracts](../generated/10-data-contracts-and-models.md) | DTOs, env, Postgres schema |
| 5 | [Audit report](../audit/audit-report.md) | Confirmed / corrected / gaps |
| 6 | [Open questions](../audit/open-questions.md) | Backlog валидации |

**Диаграммы:** [System context](../legends/system-context.md) → [Cross-area runbook](../legends/cross-area-runbook.md).

---

## Backend engineer

**Цель:** интеграция, execution paths, API.

| # | Документ | Зачем |
|---|----------|-------|
| 1 | [Cheat sheet](cheat-sheet.md) | Быстрая карта |
| 2 | [System architecture](../architecture/system-architecture.md) | Container map |
| 3 | [API & integration](../operations/api.md) | `POST /chat/agent`, health |
| 4 | [Agent runtime](../runtime/agent-runtime.md) | `iter_agent_events` |
| 5 | [Deep dive](../generated/14-deep-dive-priority-areas.md) | 6 critical paths |
| 6 | [Memory](../memory/session-memory.md) | `session_id`, UI vs server |
| 7 | [Ingestion worker](../ingestion/worker.md) + [pipeline](../ingestion/pipeline.md) | Async jobs |
| 8 | [Troubleshooting map](troubleshooting-map.md) | Incident routing |

**Smoke:** `scripts/smoke_chat_agent.ps1`, `scripts/smoke_m7.ps1` — см. [ops](../operations/ops-and-risks.md).

---

## ML / AI engineer

**Цель:** retrieval quality, chunking, tool behavior.

| # | Документ | Зачем |
|---|----------|-------|
| 1 | [similarity_search](../retrieval/similarity-search.md) | Threshold, Qdrant, embed |
| 2 | [search_kb](../retrieval/search-kb.md) | Tool contract |
| 3 | [Agent runtime](../runtime/agent-runtime.md) | Prompt, tool loop |
| 4 | [Memory](../memory/session-memory.md) | Context assembly |
| 5 | [Ingestion pipeline](../ingestion/pipeline.md) | chunk(220), embed batch |
| 6 | [Retrieval legend](../legends/retrieval-similarity-search.md) | Diagram walkthrough |

**Audit:** C3–C5 (embed path), R10 (no local fallback on agent path).

---

## Ops / support engineer

**Цель:** health, compose, jobs, incident response.

| # | Документ | Зачем |
|---|----------|-------|
| 1 | [Ops & risks](../operations/ops-and-risks.md) | Compose, smoke, risk register |
| 2 | [Troubleshooting map](troubleshooting-map.md) | «Куда смотреть если…» |
| 3 | [Cross-area runbook](../legends/cross-area-runbook.md) | Query + ingest paths, failures |
| 4 | [API health endpoints](../operations/api.md) | `/health/ready`, `/health/llm` |
| 5 | [Ingestion worker](../ingestion/worker.md) | `running` / `failed` jobs |
| 6 | [Ops & risks](../operations/ops-and-risks.md) | Compose, smoke concepts |

**Confirmed:** readiness HTTP 200 even when `degraded` (audit C22).

---

## Новый участник команды (junior / mid)

**Цель:** за 1–2 дня понять систему без чтения всего `generated/`.

| День | Документы |
|------|-----------|
| **День 1** | [index.md](../index.md) → [Cheat sheet](cheat-sheet.md) → [Executive overview](../overview/executive-overview.md) → [Architecture](../architecture/system-architecture.md) |
| **День 2** | [Agent runtime](../runtime/agent-runtime.md) → [API](../operations/api.md) → [Legends index](../legends/) |
| **День 3** | [Deep dive](../generated/14-deep-dive-priority-areas.md) → [Troubleshooting](troubleshooting-map.md) |

**Практика:** поднять stack (`scripts/quick_start.ps1`), открыть `/ui`, прогнать `smoke_chat_agent.ps1`.

---

## Executives / product (кратко)

1. [Executive overview](../overview/executive-overview.md)
2. [PRD](../overview/prd.md) — In-scope / Out-of-scope
3. [Open questions](../audit/open-questions.md) — roadmap gaps
4. [Open questions](../audit/open-questions.md) — top risks

---

## Связанные страницы

- [Troubleshooting map](troubleshooting-map.md)
- [Cheat sheet](cheat-sheet.md)
- [Generated README](../generated/README.md)
