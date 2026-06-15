# Ingestion Worker

→ **[generated/08-ingestion-worker.md](../generated/08-ingestion-worker.md)**

`python -m ingestion.worker` — Postgres queue, `FOR UPDATE SKIP LOCKED`, poll 2s. Statuses: `queued` / `running` / `done` / `failed`.

[← Ingestion](index.md) · [Troubleshooting: stale running](../navigation/troubleshooting-map.md)
