# High-level project structure (public overview)

Sanitized folder map for **ea-agent-platform**. Describes **layers and responsibilities** only — no source code in this documentation repository.

Date: `2026-06-07`.

---

## Repository layout (conceptual)

```text
ea-agent-platform/          ← implementation repository (private / separate)
├── app/                    ← HTTP API, SSE, dev UI shell
├── orchestration/          ← agent runtime, legacy multi-role graphs
├── prompts/                ← system prompts (not published)
├── parsers/                ← LLM response parsing
├── tools/                  ← tool registry, builtin tools (e.g. search_kb)
├── retrieval/              ← similarity search, citations
├── memory/                 ← session conversation store
├── ingestion/              ← ETL pipeline, background worker entrypoints
├── embeddings/             ← embedding provider adapter
├── vectorstore/            ← vector database client
├── storage/                ← object storage, job queue, local FS helpers
├── llm/                    ← chat runtime routing (vLLM / LM Studio)
├── core/                   ← shared contracts, readiness checks
├── connectors/             ← planned external sources (SQL, Confluence)
├── configs/                ← configuration templates without credentials
├── infra/                  ← deployment artifacts (not in public docs)
├── scripts/                ← smoke and ops scripts (not in public docs)
├── tests/                  ← automated tests (not in public docs)
└── docs/                   ← technical documentation (exported to this repo)
```

---

## Layer map (architecture schema)

| Layer | Folder | Responsibility | Public doc |
|-------|--------|----------------|------------|
| API / UI | `app/` | REST, SSE, minimal web panel | [Operations](../docs/operations/api.md) |
| Agent runtime | `orchestration/` | Tool loop, streaming events | [Runtime](../docs/runtime/) |
| Tools | `tools/` | `search_kb` and registry | [Retrieval](../docs/retrieval/search-kb.md) |
| Retrieval | `retrieval/` | Vector search, citations | [similarity_search](../docs/retrieval/similarity-search.md) |
| Memory | `memory/` | Short-term session turns | [Memory](../docs/memory/) |
| Ingestion | `ingestion/` | Parse, chunk, embed, index | [Ingestion](../docs/ingestion/) |
| Embeddings | `embeddings/` | Query/document vectors | [Architecture](../docs/architecture/) |
| Vector store | `vectorstore/` | Vector index operations | [Architecture](../docs/architecture/) |
| Object storage | `storage/` | Optional sync, job queue | [Ingestion worker](../docs/ingestion/worker.md) |
| Chat LLM | `llm/` | Chat completion routing | [Runtime](../docs/runtime/agent-runtime.md) |
| Contracts | `core/` | DTOs, health aggregation | [Data contracts](../docs/generated/10-data-contracts-and-models.md) |

**Deprecated / legacy:** multi-role orchestration paths under `orchestration/` — documented as legacy in [API](../docs/operations/api.md).

---

## External runtime dependencies (generic)

| Dependency | Role |
|------------|------|
| Chat runtime service | LLM inference (e.g. vLLM-compatible API) |
| Embedding provider | Text embeddings for ingest and search |
| Vector database | Similarity search index |
| Relational database | Optional: ingest jobs, session store |
| Object storage | Optional: document sync mirror |

No hostnames, ports, or credentials are listed in this public document.

---

## Documentation map

| Need | Go to |
|------|-------|
| Start reading | [docs/index.md](../docs/index.md) |
| System context diagram | [System architecture](../docs/architecture/system-architecture.md) |
| Incident routing | [Cross-area runbook](../docs/legends/cross-area-runbook.md) |
| Folder-level detail in implementation repo | Internal engineering docs (not published) |

---

## What is deliberately omitted

- File-level listings of every module
- Prompt contents and business rule text
- Connector credentials and SQL/Confluence integration details
- Evaluation datasets and quality report JSON
- Internal model paths on developer machines

[← Repository README](../README.md) · [Documentation home](../docs/index.md)
