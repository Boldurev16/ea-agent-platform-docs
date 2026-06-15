# Легенда: System context (C4 context)

**Источник:** [03-system-architecture.md](../03-system-architecture.md) §1.

## 1. Название

Flowchart: акторы, `ea-agent-platform` API, внешние сервисы.

## 2. Назначение

Показать **кто** общается с **чем** на уровне deployment (не внутренние модульные вызовы).

## 3. Как читать

- `subgraph Actors` — люди.
- `subgraph ea` — только FastAPI app в диаграмме (worker отдельный узел).
- Стрелки API → External — runtime зависимости (не каждый запрос идёт во все сервисы).

## 4. Элементы

| Блок | Роль |
|------|------|
| `FastAPI app :8000` | `ea-app` container |
| `ingestion/worker` | `ea-worker` — отдельный процесс |
| `vLLM :8001` | Chat LLM (обычно **вне** compose) |
| `LM Studio :1234` | Embeddings HTTP |
| `Qdrant :6333` | Vector index |
| `Postgres :5432` | Jobs + optional `chat_turns` |
| `MinIO :9000` | Optional object storage |

## 5. Глоссарий

| Label | Кратко RU | Роль E2E |
|-------|-----------|----------|
| `runtime` | Chat inference | Agent answers |
| `embed` | Embedding service | Ingest + search |
| `vector store` / `Qdrant` | Индекс chunks | RAG |
| `queue` | `ingest_jobs` в Postgres | Async ingest |

## 6. Код / compose

`docker-compose.yml`, `app/api/main.py`, `ingestion/worker.py`.

**Audit R8–R9:** `app` depends_on только postgres; worker + qdrant started.

## 7. Happy path (запрос)

User → API → vLLM (chat) + Qdrant (via tool) + LM Studio (embed) + Postgres (memory/jobs).

## 8. Ошибки

- «App depends on all backends» — **неверно** (R8).
- «vLLM в compose» — обычно external (Inferred из `environment configuration template`).

## 9. Документы

[03-system-architecture.md](../03-system-architecture.md), [12-ops-observability-and-risks.md](../12-ops-observability-and-risks.md).
