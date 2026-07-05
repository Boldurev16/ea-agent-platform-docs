# Similarity search: смысловой поиск по базе знаний

`similarity_search` — ядро retrieval-контура. Он не ищет буквальное совпадение слов, а сравнивает смысл вопроса с фрагментами документов через embeddings.

## Что происходит

```mermaid
flowchart LR
  query["Вопрос"] --> embed["Embedding запроса"]
  embed --> qdrant["Поиск в vector store"]
  qdrant --> filter["Фильтр качества"]
  filter --> citations["Фрагменты и источники"]
```

## Что должен понимать архитектор

- Результат поиска — это набор фрагментов, а не весь документ.
- Чем лучше подготовлен corpus, тем выше качество ответа.
- `similarity_threshold` отсекает слабые совпадения.
- Citations помогают проверить, на какие источники опирается агент.

## Детальный разбор

См. [generated/07-retrieval-similarity_search.md](../generated/07-retrieval-similarity_search.md).

[← Retrieval](index.md) · [Legend](../legends/retrieval-similarity-search.md)
