# 10 — Data Contracts and Models

## Overview

Контракты разделены на:
- **Pydantic DTOs** — `core/contracts.py`, `retrieval/contracts.py`
- **Protocol interfaces** — `core/provider_protocols.py`
- **Runtime dicts** — agent steps, ingest reports, job rows
- **Postgres schema** — `storage/postgres_client.py`
- **Qdrant payload** — implicit in `vectorstore/qdrant_store.py`

## Pydantic Models (`core/contracts.py`)

### `RuntimeProviderHealth`

| Field | Type | Meaning |
|-------|------|---------|
| `provider` | str | `lmstudio` \| `vllm` |
| `status` | RuntimeStatus | not-enabled \| configured \| unreachable \| healthy |
| `configured` | bool | Env present |
| `enabled` | bool | Feature flag |
| `base_url` | str | Endpoint |
| `model` | str | Model id |
| `reachable` | bool | HTTP ok |
| `model_available` | bool | Model loaded |
| `last_error` | str | Error text |
| `action_hint` | str | Ops hint |

**Used by:** documentation target; `/health/llm` returns dict, not validated model — **Inferred**.

### `Citation`

| Field | Type | Meaning |
|-------|------|---------|
| `source_uri` | str | Display source (filename) |
| `object_key` | str | MinIO-style key |
| `chunk_id` | str | Chunk identifier |
| `page` | int \| None | Optional page |
| `section` | str | Optional section |

**Used by:** `RetrievalResponse`, `search_kb` output, SSE citations.

### `RetrievedChunk`

| Field | Type | Meaning |
|-------|------|---------|
| `text` | str | Chunk text |
| `score` | float | Similarity score |
| `source` | str | e.g. `vector` |
| `metadata` | dict | Qdrant payload fields |
| `citation` | Citation \| None | Linked citation |

### `RetrievalRequest`

| Field | Type | Default | Validation |
|-------|------|---------|------------|
| `query` | str | — | min_length=1 |
| `corpus_id` | str | `default` | — |
| `agent_scope` | str | `default` | **Not used in similarity_search** |
| `top_k` | int | 5 | 1–50 |
| `filters` | dict | {} | **Not applied in search** |
| `include_debug` | bool | True | — |

### `RetrievalResponse`

| Field | Type |
|-------|------|
| `query` | str |
| `chunks` | list[RetrievedChunk] |
| `citations` | list[Citation] |
| `debug` | dict |

### `DocumentUploadRequest` / `IngestRequest`

Planned API shapes — **partially used**:
- `IngestRequest` fields `object_keys`, `chunkers`, `force_reindex` — **not wired** to `POST /ingestion/run` — Confirmed gap
- `DocumentUploadRequest` — MinIO upload path planned

### `SimilaritySearchRequest` (`retrieval/contracts.py`)

Extends `RetrievalRequest` with:
| Field | Default |
|-------|---------|
| `similarity_threshold` | 0.25 (0.0–1.0) |
| `filter_identifiers` | [] |

## Agent Step JSON (runtime, not Pydantic)

Parsed by `parsers/agent_response.py::parse_agent_step`:

| action | Required fields |
|--------|-----------------|
| `tool` | `name`, `arguments` (dict) |
| `final` | `answer`, optional `citations` |

## SSE Event Payloads (`orchestration/agent_stream.py`)

| event | data keys |
|-------|-----------|
| `status` | `phase`, `session_id` |
| `tool_call` | `tool`, `arguments` |
| `sources` | `citations` |
| `text` | `chunk` |
| `introspect` | `trace`, `tool_calls`, `message` |
| `close` | `status`, `answer`, `citations`, `tool_calls`, `provider`, `model`, `parse_fallback`, `session_id` |

Formatted by `app/api/sse.py::format_sse`.

## Postgres Tables

### `ingest_jobs`

| Column | Type | Meaning |
|--------|------|---------|
| `job_id` | UUID PK | Job identifier |
| `status` | VARCHAR(32) | queued \| running \| done \| failed |
| `corpus_id` | VARCHAR(128) | Target corpus |
| `created_at` | TIMESTAMPTZ | Created |
| `updated_at` | TIMESTAMPTZ | Updated |
| `report` | JSONB | Full ingest report on success |
| `error` | TEXT | Error message on failure |

### `chat_turns`

| Column | Type | Meaning |
|--------|------|---------|
| `session_id` | VARCHAR(128) | Session |
| `turn_index` | INT | Order |
| `query` | TEXT | User |
| `answer` | TEXT | Assistant |
| `created_at` | TIMESTAMPTZ | — |

## Qdrant Point Payload

| Field | Source |
|-------|--------|
| `chunk_id` | chunk dict |
| `corpus_id` | ingest param |
| `source_file` | chunk dict |
| `order` | chunk dict |
| `text` | chunk dict |

Point ID: `uuid5(NAMESPACE_URL, chunk_id)` — `vectorstore/qdrant_store.py::point_id_for_chunk`

## Chunk Dict (ingest)

| Field | Type |
|-------|------|
| `chunk_id` | str |
| `source_file` | str |
| `order` | int |
| `text` | str |

## Environment Variables (complete table)

See `runtime_settings.py` + `core/retry_policy.py`:

| Variable | Default | Accessor |
|----------|---------|----------|
| `RUNTIME_PROVIDER` | lmstudio | `get_runtime_provider` |
| `VLLM_BASE_URL` | <chat-runtime-host>:<chat-port>/v1 | `get_vllm_base_url` |
| `VLLM_MODEL` | <chat-model-id> | `get_vllm_model` |
| `LMSTUDIO_BASE_URL` | <embedding-host>:<embed-port>/v1 | `get_lmstudio_base_url` |
| `LMSTUDIO_EMBED_MODEL` | nomic v1.5 | `get_lmstudio_embed_model` |
| `LMSTUDIO_EMBED_FALLBACK_ENABLED` | false | `get_lmstudio_embed_fallback_enabled` |
| `QDRANT_HOST/PORT/COLLECTION` | <vector-store-host>, <vector-store-port>, <vector-collection-name> | various |
| `CORPUS_ID_DEFAULT` | default | `get_default_corpus_id` |
| `STORAGE_PROVIDER` | auto | `get_storage_provider` |
| `VECTORSTORE_PROVIDER` | qdrant | `get_vectorstore_provider` |
| `EMBEDDING_PROVIDER` | nomic | `get_embedding_provider` |
| `POSTGRES_*` | see environment configuration template | postgres_client |
| `SESSION_STORE` | memory | `get_session_store` |
| `INGEST_ASYNC_DEFAULT` | true | `get_ingest_async_default` |
| `AGENT_MAX_TOOL_CALLS` | 5 | `get_agent_max_tool_calls` |
| `AGENT_MAX_TOKENS` | 1024 | `get_agent_max_tokens` |
| `RETRIEVAL_LOCAL_FALLBACK_ENABLED` | false | `get_retrieval_local_fallback_enabled` |
| `RETRY_ATTEMPTS` | 3 | `get_retry_attempts` |
| `RETRY_BACKOFF_SECONDS` | 0.5 | `get_retry_backoff_seconds` |

## Provider Protocols (`core/provider_protocols.py`)

| Protocol | Key methods |
|----------|-------------|
| `StorageProvider` | `sync_sources` |
| `VectorStoreProvider` | `upsert_embeddings`, `search_vectors`, `collection_exists`, `ping` |
| `EmbeddingProvider` | `embed_texts`, `embed_query`, `health` |
| `ChatProvider` | `generate_chat_completion`, `health` |

## API Request Models (`app/api/main.py`)

| Model | Fields |
|-------|--------|
| `QueryRequest` | `query`, `include_debug`, `role_override` |
| `ChatAgentRequest` | `query`, `session_id`, `include_debug` |
| `KnowledgeDocumentRequest` | `filename`, `content` |
| `AgentPanelRequest` | `query`, `include_debug`, `role_override` |

## Evidence

- `core/contracts.py`
- `retrieval/contracts.py`
- `storage/postgres_client.py`
- `runtime_settings.py`
- `app/api/main.py`
