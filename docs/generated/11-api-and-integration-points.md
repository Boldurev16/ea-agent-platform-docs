# 11 — API and Integration Points

Base URL (default): `<application-base-url>`

**Authentication:** none — Confirmed gap.

## Health & Ops

| Method | Path | Response | Module |
|--------|------|----------|--------|
| GET | `/health` | `{"status":"ok"}` | `core/readiness.py::check_liveness` |
| GET | `/health/live` | same | `app/api/main.py` |
| GET | `/health/ready` | `{status, dependencies, blocking}` | `core/readiness.py::check_readiness` |
| GET | `/health/llm` | runtime + embeddings snapshot | `llm/runtime_health.py` |

### Readiness dependencies checked

- `postgres` (if enabled)
- `qdrant` (vectorstore ping)
- `embeddings` (Nomic health)
- `storage` (if `STORAGE_PROVIDER=minio`)

## Primary Agent APIs

### `POST /chat/agent` — **Primary integration**

| Aspect | Detail |
|--------|--------|
| Content-Type | `application/json` request |
| Response | `text/event-stream` |
| Body | `ChatAgentRequest`: `query`, `session_id` (default `default`), `include_debug` (default true) |

**SSE events:** `status`, `tool_call`, `sources`, `text`, `introspect`, `close`

**Handler:** `app/api/main.py::chat_agent` → `iter_agent_events` → `format_sse`

**Example:**
```http
POST /chat/agent
Content-Type: application/json

{"query": "Что в техрадаре EA?", "session_id": "user-1", "include_debug": true}
```

### `POST /tasks/agent` — Sync JSON

Вызывает `run_agent(payload.query)` — **без** `session_id`; server memory всегда `session_id="default"`.

Returns:
```json
{
  "answer": "...",
  "citations": [...],
  "trace": [...],
  "tool_calls": 1,
  "status": "completed",
  "provider": "vllm",
  "model": "..."
}
```

**Use case:** tests, simple integrations without SSE.

## Knowledge Base & Ingestion

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/kb/documents` | Write text file to `data/raw/{filename}` |
| POST | `/ingestion/run` | Start ingest (`async_mode` query param, `corpus_id`) |
| GET | `/ingestion/jobs/{job_id}` | Job status |

### Async ingest response (default `INGEST_ASYNC_DEFAULT=true`)

```json
{
  "job_id": "uuid",
  "status": "queued",
  "corpus_id": "default",
  "async": true
}
```

### Sync ingest (`async_mode=false`)

Returns full `ingestion_report` dict from `run_ingest_pipeline`.

## Legacy APIs (deprecated)

| Method | Path | Notes |
|--------|------|-------|
| POST | `/tasks/orchestrate` | `legacy_deprecated: true` на **корне** response |
| POST | `/tasks/panel` | pre-RAG + 4 roles; `legacy_deprecated` только в nested `orchestrator` |
| POST | `/tasks/role/{architect,auditor,economist,analyst}` | pre-RAG via `document_retrieve` |
| GET | `/tasks/orchestrate/debug-schema` | JSON schema for legacy orchestrate response |

**Recommendation:** migrate to `/chat/agent`.

**Memory:** нет HTTP endpoint для очистки server-side session memory. UI «Очистить историю» очищает только localStorage `dialogTurns`, не `chat_turns` / `ShortTermMemory`.

## UI

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/ui` | HTML dev panel |
| GET | `/ui.js` | Embedded JS (SSE client, ingest poll) |

## External Integration Dependencies

Integrators must ensure:

| Service | Required for |
|---------|--------------|
| vLLM | Chat/agent (`RUNTIME_PROVIDER=vllm`) |
| LM Studio | Embeddings (ingest + search_kb) |
| Qdrant | Retrieval |
| Postgres | Async ingest queue, optional sessions |
| MinIO | Optional object storage sync |

## Integration Patterns

### SSE client

1. `POST /chat/agent` with `Accept: text/event-stream`
2. Parse lines `event:` / `data:` JSON
3. Accumulate `text` chunks until `close`
4. Read `citations` from `sources` or `close.data`

**Reference:** `app/api/sse.py`, UI JS in `main.py`.

### Async ingest polling

1. `POST /ingestion/run`
2. Loop `GET /ingestion/jobs/{job_id}` until `status` in `done|failed`

**Reference:** UI `pollIngestJob` in `main.py`.

## Error Responses

- FastAPI validation errors — 422 for invalid body
- Agent LLM down — SSE `close` with `status=error` (not HTTP 5xx)
- Readiness degraded — HTTP 200 with `status: degraded` — **Confirmed** (not 503)

## Evidence

- `app/api/main.py`
- `tests/test_user_interface_endpoints.py`
- `tests/test_chat_agent_stream.py`

## Open Questions

- **Needs verification:** CORS configuration for embed/widget (not present)
- Rate limiting — not implemented
