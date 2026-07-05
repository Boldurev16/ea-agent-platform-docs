# Легенда: search_kb sequence

**Источник:** [06-tools-search_kb.md](../06-tools-search_kb.md), [14-deep-dive](../14-deep-dive-priority-areas.md) §3.

## 1. Название

Sequence: `agent_stream` → `ToolRegistry` → `search_kb` → `perform_similarity_search` → embed → Qdrant.

## 2. Назначение

Полный call chain tool **от agent loop до vector store** (включая embed).

## 3. Как читать

- `search_kb` — thin wrapper; ranking в `similarity_search.search`.
- `EmbeddingProvider` — обязательный шаг перед Qdrant.
- Return path: `RetrievalResponse` → dict → JSON в `messages` role `tool`.

## 4. Глоссарий

| Label | Кратко RU | Где | Роль |
|-------|-----------|-----|------|
| `search_kb` | Tool handler | `tools/builtin/search_kb.py` | Agent RAG entry |
| `similarity_search` | Module | `retrieval/similarity_search.py` | Core algorithm |
| `perform_similarity_search` | Public API | Wrapper → `search()` | Params → request |
| `embed_query` | Query embedding | Nomic prefix `search_query:` | Vector for search |
| `search_vectors` | Запрос в Qdrant | Top-K по cosine similarity | Первичные результаты поиска |
| `similarity_threshold` | Min score | Default 0.25 | Filter |
| `top_k` | Max hits | Default 5 | Qdrant limit |
| `citations` | Source refs | `build_citation` | SSE `sources` |
| `hits` | Tool output | Mapped chunks | LLM context |

## 5. Код

```
ToolRegistry.call("search_kb", ...)
  → search_kb()
  → perform_similarity_search()
  → search(SimilaritySearchRequest)
  → embed_query + search_vectors
```

## 6. Happy path

Agent calls tool with query → embed → Qdrant top_k → filter threshold → citations in tool_result → LLM final answer.

## 7. Ошибки понимания

| Ошибка | Факт |
|--------|------|
| Local `chunks.json` fallback on agent path | **Нет** — только `SimilaritySearchError` (R10, C21) |
| `retrieve_vector` same as search_kb | search_kb → `perform_similarity_search` only (C2) |
| `vector_retrieve` in agent | **Не используется** в agent registry |

Legacy: `document_retrieve` / `retrieve_vector` — отдельные paths (pre-RAG, tests).

## 8. Документы

[06-tools-search_kb.md](../06-tools-search_kb.md), [07-retrieval-similarity_search.md](../07-retrieval-similarity_search.md), [retrieval-similarity-search-legend-ru.md](retrieval-similarity-search-legend-ru.md).

## 9. Ambiguities

- U5: `LMSTUDIO_EMBED_FALLBACK_ENABLED` vs search quality
- U10: panel relevance filter ≠ agent retrieval
