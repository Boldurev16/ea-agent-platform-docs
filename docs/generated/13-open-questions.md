# 13 — Open Questions

Список неоднозначностей, пробелов и пунктов для валидации с командой. Статусы: **Confirmed gap** (код явно не реализует), **Needs verification** (не проверено в runtime), **Assumption** (логическое следование).

## Top 10 — Human Validation Required

| # | Question | Status | Evidence / Notes |
|---|----------|--------|------------------|
| 1 | Почему `ea-periodic-worker` перезапускается в compose? | Needs verification | Compose logs; `scripts/run_periodic_ingestion.py` |
| 2 | Допустимый размер batch для `embed_texts` при ~10k chunks? | Needs verification | `ingestion/pipeline.py` single call |
| 3 | Безопасно ли экспонировать API без auth в on-prem? | Assumption | No auth in `app/api/main.py` |
| 4 | Используется ли `agent_scope` / `filters` в retrieval roadmap? | Confirmed gap | DTO exists; `similarity_search` ignores |
| 5 | Когда wiring `IngestRequest.chunkers` и `force_reindex`? | Confirmed gap | DTO vs `POST /ingestion/run` |
| 6 | Horizontal scaling workers — job locking в Postgres? | **Confirmed** — `FOR UPDATE SKIP LOCKED` в `claim_next_ingest_job` | `storage/ingest_jobs.py` |
| 7 | Production target: vLLM only chat или LM Studio fallback? | Needs verification | `RUNTIME_PROVIDER` env |
| 8 | Retention policy для `chat_turns` и `ingest_jobs`? | Confirmed gap | No TTL in schema |
| 9 | Parity Phase 2 P1 — workspaces как corpus_id mapping? | Assumption | Plan doc vs current single corpus |
| 10 | Flaky test `test_iter_agent_events_emits_sources_on_tool_call` без Qdrant? | Needs verification | `tests/test_chat_agent_stream.py` |

## Agent Runtime

| Question | Status |
|----------|--------|
| Streaming token-by-token from vLLM vs chunked `text` events? | Needs verification — `agent_stream.py` may batch |
| Max context window enforcement beyond `AGENT_MAX_TOKENS`? | Partial — generation limit only |
| Corporate architect prompt versioning strategy? | Not in code |

## Memory

| Question | Status |
|----------|--------|
| Default `SESSION_STORE=memory` in multi-replica deploy? | Ops risk — use postgres |
| Summarization of long sessions? | Confirmed gap |
| Cross-session long-term memory? | Confirmed gap |

## Tools (`search_kb`)

| Question | Status |
|----------|--------|
| Authorization per corpus/workspace? | Confirmed gap |
| Rate limit per session on tool calls? | Only `AGENT_MAX_TOOL_CALLS` global |

## Retrieval

| Question | Status |
|----------|--------|
| Optimal `similarity_threshold` 0.25 for Russian EA docs? | Needs validation |
| Hybrid search (BM25 + vector)? | Confirmed gap |
| Re-ranking model? | Confirmed gap |

## Ingestion

| Question | Status |
|----------|--------|
| Semantic chunking vs fixed 220 chars? | Confirmed gap — fixed only |
| Incremental ingest by file hash? | Confirmed gap — full reprocess |
| Confluence / SQL sources (Phase 2 P2)? | Planned, not in code |

## API & Product

| Question | Status |
|----------|--------|
| Official deprecation timeline for `/tasks/*`? | Not documented in code |
| CORS / embed widget API? | Confirmed gap |
| OpenAPI export for external teams? | FastAPI auto — **Needs verification** `/docs` |

## Infrastructure

| Question | Status |
|----------|--------|
| Helm chart production-ready vs compose-only dev? | Needs verification |
| NOTICE.md license obligations for bundled models? | See `NOTICE.md` |
| MinIO required in prod or local-only ingest path? | `STORAGE_PROVIDER=auto` |

## Observability

| Question | Status |
|----------|--------|
| Prometheus metrics endpoint? | Confirmed gap |
| Distributed tracing (OpenTelemetry)? | Confirmed gap |
| Alerting on `ingest_jobs.failed`? | Confirmed gap |

## Data & Compliance

| Question | Status |
|----------|--------|
| PII handling in `data/raw/`? | Not in code |
| Audit log for KB document uploads? | Confirmed gap |
| Encryption at rest for Postgres/Qdrant? | Infra-level |

## Resolution Process

1. Owner assigns each ID to engineer or architect.
2. Validate in staging with smoke suite + manual queries.
3. Update this file and linked module docs when resolved.
4. Track product gaps in the private implementation repository planning documents.

## Evidence

- Entire `ea-agent-platform` codebase scan
- Private implementation repository milestone reports
- `plans/ea_rag_consolidation_355c452a.plan.md`
