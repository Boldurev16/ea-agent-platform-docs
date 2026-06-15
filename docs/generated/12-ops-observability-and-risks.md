# 12 — Operations, Observability and Risks

## Deployment Topology

### Docker Compose (`docker-compose.yml`)

| Service | Image / build | Role |
|---------|---------------|------|
| `ea-app` | `ea-agent-platform` | FastAPI on `:8000` |
| `ea-worker` | same | `ingestion/worker.py` poll consumer |
| `ea-periodic-worker` | same | `scripts/run_periodic_ingestion.py` (300s loop) |
| `ea-postgres` | postgres:16 | Jobs + optional sessions |
| `ea-qdrant` | qdrant | Vector index |
| `ea-minio` | minio | Optional object storage |

**Evidence:** `docker-compose.yml`, `infra/helm/ea-agent-platform/`.

### Helm profiles

- `values-dev.yaml`, `values-on-prem.yaml`, `values-k8s.yaml`
- **Needs verification:** full chart templates vs compose parity

## Processes & Background Jobs

| Process | Trigger | Work |
|---------|---------|------|
| `ea-worker` | Poll `ingest_jobs` | `run_ingest_pipeline` |
| `ea-periodic-worker` | Timer 300s | inbox → raw + pipeline |
| In-app | HTTP | Agent, sync ingest |

**Known issue:** `ea-periodic-worker` restart loop in compose — **Needs verification** root cause.

## Health Checks

| Endpoint | Use |
|----------|-----|
| `GET /health/live` | Liveness — process up |
| `GET /health/ready` | Readiness — dependency matrix |
| `GET /health/llm` | Chat + embed provider snapshot |

Readiness returns `blocking: true` when critical deps fail — orchestrator can use for scheduling.

**Evidence:** `core/readiness.py` (нет отдельного `tests/test_health_endpoints.py` в репозитории).

## Logging

| Area | Mechanism |
|------|-----------|
| FastAPI / uvicorn | Standard access logs |
| Agent stream | `introspect` SSE event with trace |
| Worker | Python logging in `ingestion/worker.py` |
| Structured metrics | **Not implemented** — Confirmed gap |

**Inferred:** no centralized log aggregation configured in repo.

## Smoke Test Suite

| Script | Validates |
|--------|-----------|
| `scripts/smoke_compose.py` | Containers healthy |
| `scripts/smoke_ingest.py` | Pipeline |
| `scripts/smoke_retrieval.py` | similarity_search |
| `scripts/smoke_agent_live.py` | Agent against live stack |
| `scripts/smoke_chat_agent.py` | SSE stream |
| `scripts/smoke_m7.py` | Async ingest + readiness |

PowerShell wrappers: `scripts/smoke_*.ps1`

## Retry & Resilience

| Component | Policy |
|-----------|--------|
| HTTP clients | `core/retry_policy.py` — `RETRY_ATTEMPTS`, `RETRY_BACKOFF_SECONDS` |
| Embed fallback | `LMSTUDIO_EMBED_FALLBACK_ENABLED` — dev compose often `true` |
| Retrieval fallback | `RETRIEVAL_LOCAL_FALLBACK_ENABLED` — default false |
| Ingest job | Worker marks `failed` on exception — no automatic retry queue |

**Evidence:** `core/retry_policy.py`, `storage/ingest_jobs.py`.

## Operational Runbooks (derived)

### Start stack
```bash
docker compose up -d
```

### Rebuild after code change
```bash
docker compose build ea-app ea-worker
docker compose up -d ea-app ea-worker
```

### Manual sync ingest
```http
POST /ingestion/run?async_mode=false&corpus_id=default
```

### Check KB size
- Qdrant collection `<vector-collection-name>` point count
- `data/chunks.json` length

## Security Posture

| Topic | Status |
|-------|--------|
| API authentication | **Not implemented** |
| TLS | External to app (ingress) |
| Secrets in env | `environment configuration template` — no vault integration in code |
| MinIO credentials | Compose env |
| Postgres | Compose env |
| Prompt injection | No dedicated guardrails beyond tool loop limit |

**Risk:** public `/chat/agent` and `/kb/documents` in dev compose.

## Performance Characteristics

| Path | Bottleneck |
|------|------------|
| Agent query | vLLM latency + tool rounds (max `AGENT_MAX_TOOL_CALLS`) |
| search_kb | Embed query + Qdrant search |
| Full ingest | Embed all chunks single batch |
| Session memory | In-memory or Postgres per turn |

**No** explicit SLA targets in code.

## Scalability Limits

| Dimension | Limit |
|-----------|-------|
| App replicas | Session store must be `postgres` for shared sessions |
| Workers | Single consumer poll — **no** horizontal partition in code |
| Qdrant | Single collection per corpus_id filter |
| Corpus size | Limited by embed batch memory |

## Risk Register

| ID | Risk | Severity | Mitigation in code |
|----|------|----------|-------------------|
| R1 | No auth on API | High | Network isolation only |
| R2 | LM Studio embed down blocks ingest/search | High | Optional embed fallback flag |
| R3 | Full re-ingest cost on every run | Medium | Idempotent upsert by chunk_id |
| R4 | `IngestRequest` fields ignored | Medium | Document + future wiring |
| R5 | Legacy endpoints confuse integrators | Low | `legacy_deprecated` flag |
| R6 | periodic-worker unstable | Medium | Ops fix needed |
| R7 | Single-threaded ingest pipeline per job | Medium | Scale worker replicas; SKIP LOCKED supports multi-worker |
| R8 | No metrics/tracing | Medium | Add Prometheus/OpenTelemetry |
| R9 | Agent JSON parse failures | Medium | `parse_fallback` in close event |
| R10 | Qdrant unavailable → empty retrieval | High | Readiness check; no graceful answer |

## Backup & DR

**Not implemented in codebase:**
- Qdrant backup automation
- Postgres backup
- MinIO replication

**Assumption / Needs verification:** ops procedures external to repo.

## Evidence

- `docker-compose.yml`
- `runtime_settings.py`
- `core/readiness.py`
- `storage/ingest_jobs.py`
- `NOTICE.md`
