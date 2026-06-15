# Легенда: similarity_search (retrieval)

**Источник:** [07-retrieval-similarity_search.md](../07-retrieval-similarity_search.md), [14-deep-dive](../14-deep-dive-priority-areas.md) §4.

## 1. Название

- Flowchart: query → embed → Qdrant → filter → `RetrievalResponse`
- Sequence: caller → `perform_similarity_search` → providers → Qdrant

## 2. Назначение

Внутренняя логика vector retrieval без agent/tool обёртки.

## 3. Как читать flowchart LR

Линейный pipeline внутри `search()` — каждый блок = функция или filter step.

## 4. Глоссарий

| Label | Кратко RU | Роль |
|-------|-----------|------|
| `query` | Текст запроса | Input |
| `EmbeddingProvider.embed_query` | Вектор запроса | LM Studio HTTP |
| `QdrantVectorStoreProvider.search_vectors` | Vector search | Top-K |
| `threshold` | `similarity_threshold` | Post-filter scores |
| `filter_identifiers` | Exclude list | Skip matching source ids |
| `curate_sources` | Metadata extract | `retrieval/citations.py` |
| `build_citation` | Citation DTO | UI/SSE |
| `RetrievalResponse` | Pydantic result | chunks + citations + debug |
| `empty_retrieval_response` | No collection / no hits | Not exception |

## 5. Код

`retrieval/similarity_search.py::search`, `perform_similarity_search`.

Callers:
- `tools/builtin/search_kb.py`
- `retrieval/document_retriever.py` (legacy)
- `retrieval/vector_retriever.py` (wraps + optional fallback)

## 6. Happy path

Collection exists → embed_query → search_vectors → filter ≥0.25 → build chunks/citations → debug with `latency_ms`.

## 7. Ошибки

| Ошибка | Факт |
|--------|------|
| Fallback в similarity_search | **Нет** — raises `SimilaritySearchError` |
| Fallback при ошибке | `vector_retriever` + env only (C21) |
| `RetrievalRequest.filters` | **Не применяются** в search() |
| BM25 / hybrid | `hybrid_retrieve` = alias `document_retrieve` (C27) |

## 8. Документы

[07-retrieval-similarity_search.md](../07-retrieval-similarity_search.md), [10-data-contracts](../10-data-contracts-and-models.md).

## 9. Ambiguities

U5 (embed fallback), U8 (flaky tests without Qdrant).
