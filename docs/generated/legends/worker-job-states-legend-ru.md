# Легенда: Worker job states + enqueue sequence

**Источник:** [08-ingestion-worker.md](../08-ingestion-worker.md), [14-deep-dive](../14-deep-dive-priority-areas.md) §5.

## 1. Название

- State: `ingest_jobs` lifecycle (`queued` → `running` → `done`|`failed`)
- Sequence: `POST /ingestion/run` → poll worker → pipeline

## 2. Назначение

Async ingest: как job попадает в очередь и как worker его обрабатывает.

## 3. Как читать stateDiagram

| State | Postgres `status` |
|-------|-------------------|
| `queued` | Ждёт worker |
| `running` | После `claim_next_ingest_job` |
| `done` | Успех + `report` JSONB |
| `failed` | Exception + `error` text |

**Не** `completed` в Postgres API (audit R12).

## 4. Sequence diagram

| Участник | Роль |
|----------|------|
| `POST /ingestion/run` | Enqueue |
| `ingest_jobs table` | Postgres queue |
| `ingestion/worker` | Poll consumer |
| `run_ingest_pipeline` | Sync ETL |

Loop `poll every 2s` — `run_worker_loop(poll_seconds=2.0)`.

## 5. Глоссарий

| Label | Кратко RU | Роль |
|-------|-----------|------|
| `Op` / Operator | Оператор | Триггер ingest |
| `POST /ingestion/run` | Start ingest | `enqueue_ingest_job` |
| `worker` | `ea-worker` container | Consumer |
| `queue` / `ingest_jobs` | Job table | Async boundary |
| `job` / `job_id` | UUID задачи | Poll + status API |
| `FOR UPDATE SKIP LOCKED` | Multi-worker safe claim | `claim_next_ingest_job` |
| `pipeline` | Full corpus ingest | Per job |

## 6. Код

- `ingestion/worker.py::process_next_job`, `run_worker_loop`
- `storage/ingest_jobs.py`
- `app/api/main.py::ingestion_run`, `ingestion_jobs_get`

## 7. Happy path

API enqueue → worker claim → `run_ingest_pipeline(corpus_id, job_id)` → `complete_ingest_job` → client polls `GET /ingestion/jobs/{id}` → `done`.

## 8. Ошибки / open questions

| Тема | Статус |
|------|--------|
| Stale `running` после crash | Confirmed gap U3 |
| Worker re-raise kills process | Needs verification U2 |
| Auto-retry failed jobs | **Не реализовано** |
| JSONL fallback + worker | Worker **требует** Postgres healthy |

## 9. Документы

[08-ingestion-worker.md](../08-ingestion-worker.md), [11-api](../11-api-and-integration-points.md), audit C12–C13, R16, U2–U3.
