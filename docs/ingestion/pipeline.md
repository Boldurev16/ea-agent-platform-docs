# Ingestion pipeline

Ingestion pipeline — это производственный контур включения документов в корпоративную базу знаний. Его задача — сделать документы пригодными для смыслового поиска: разобрать, нормализовать, разделить на фрагменты, построить embeddings и индексировать.

## Почему это важно

Агент не “читает папку с файлами” напрямую. Он работает с подготовленной базой знаний. Поэтому качество ingestion напрямую влияет на качество ответов.

```mermaid
flowchart LR
  docs["Документы"] --> parse["Разбор"]
  parse --> normalize["Нормализация"]
  normalize --> chunks["Фрагменты"]
  chunks --> embeddings["Embeddings"]
  embeddings --> index["Индекс"]
  index --> kb["База знаний"]
```

## Детальный разбор

См. [generated/09-ingestion-pipeline.md](../generated/09-ingestion-pipeline.md).

[← Ingestion](index.md) · [Legend](../legends/ingestion-pipeline.md)
