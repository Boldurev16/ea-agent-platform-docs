# API и интеграционные точки

API платформы делится на три группы: пользовательский чат, управление базой знаний и эксплуатационная диагностика.

| Контур | Endpoint | Назначение |
|--------|----------|------------|
| Чат | `POST /chat/agent` | Основной потоковый ответ агента. |
| Ingestion | `POST /ingestion/run` | Поставить задачу обновления базы знаний. |
| Ingestion | `GET /ingestion/jobs/{job_id}` | Проверить статус обработки. |
| Health | `GET /health/live` | Проверить, что приложение живо. |
| Health | `GET /health/ready` | Проверить готовность зависимостей. |
| Health | `GET /health/llm` | Проверить контур LLM и embeddings. |

Legacy endpoint остаются для совместимости, но не являются основным пользовательским путем.

Детальный справочник: [generated/11-api-and-integration-points.md](../generated/11-api-and-integration-points.md).

[← Operations](index.md)
