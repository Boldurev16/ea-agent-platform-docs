# Легенда: Ingestion pipeline

**Источник:** [03-system-architecture.md](../03-system-architecture.md) §3.2, [09-ingestion-pipeline.md](../09-ingestion-pipeline.md), [14-deep-dive](../14-deep-dive-priority-areas.md) §6.

## 1. Название

Flowchart: `data/raw/` → parse → chunk(220) → embed → Qdrant → artifacts.

## 2. Назначение

End-to-end индексация документов в vector store и файлы отчёта.

## 3. Как читать

- Нумерованная версия (09) — **точный порядок** в `run_ingest_pipeline`.
- Версия 03 §3.2: ветка `sync MinIO` — optional parallel визуально, в коде **до** load.
- Стрелки TD — строгая последовательность этапов.

## 4. Легенда блоков

| Блок | Функция | Модуль |
|------|---------|--------|
| `sync_sources_to_storage` | Upload raw → MinIO | `ingestion/pipeline.py` |
| `load_text_documents` | Multi-format load | `ingestion/loaders/text_loader.py` |
| `parse_documents` | Extract text | `ingestion/parsers/text_parser.py` |
| `normalize_records` | Cleanup | `ingestion/normalizers/text_normalizer.py` |
| `chunk_records max 220` | Fixed-size chunks | `ingestion/chunking/text_chunker.py` |
| `embed_texts` | Batch vectors | `embeddings/nomic_embeddings.py` |
| `upsert_embeddings` | Qdrant write | `vectorstore/qdrant_store.py` |
| `build_provenance` | Provenance JSON | `ingestion/provenance/` |
| `ingestion_report.json` | Run report | `data/` |

## 5. Глоссарий меток

| Label | Кратко RU | Роль |
|-------|-----------|------|
| `chunk(220)` | Чанк ≤220 символов | Retrieval granularity |
| `embed_texts` | Индексный embed | `search_document:` prefix |
| `pipeline` | `run_ingest_pipeline` | Orchestrator |
| `STORAGE_PROVIDER` | local/minio | MinIO sync optional |
| `complete_ingest_job` | Job done + report | Async path with `job_id` |
| `data/raw/` | Локальный read path | **Parse всегда отсюда** (R14) |

## 6. Код

`ingestion/pipeline.py::run_ingest_pipeline` — single orchestrator.

## 7. Happy path

Optional MinIO sync → load → parse → normalize → chunk → embed all texts one batch → upsert batches of 100 → provenance + report → `complete_ingest_job` if `job_id`.

## 8. Ошибки понимания

| Ошибка | Факт |
|--------|------|
| Parse из MinIO | Локальный `data/raw/` (R14) |
| Delta / skip unchanged | Full reprocess каждый run |
| `IngestRequest.chunkers` | DTO не подключён к API |

**Ambiguity U4:** batch size <environment-specific-count> chunks в одном `embed_texts`.

## 9. Документы

[09-ingestion-pipeline.md](../09-ingestion-pipeline.md), [08-ingestion-worker.md](../08-ingestion-worker.md), audit C14–C17, R14.
