# Enterprise AI Agent Platform Handbook

Документация по архитектуре агентной системы для управления корпоративной архитектурой.

Репозиторий описывает **enterprise‑контур AI‑агентов** для работы с корпоративной архитектурой:  
главный агент архитектуры, корпоративная база знаний, поиск с цитатами источников, потоковый чат, LLM‑gateway и контур загрузки/ингеста документов.

## С чего начать

- Главная страница: [`docs/index.md`](docs/index.md)
- Маршруты чтения по ролям: [`docs/navigation/reading-paths.md`](docs/navigation/reading-paths.md)
- Архитектурная шпаргалка: [`docs/navigation/cheat-sheet.md`](docs/navigation/cheat-sheet.md)
- Глоссарий терминов (EA, LLM gateway, RAG, MCP, guardrails) [`docs/navigation/glossary.md`](docs/navigation/glossary.md)

## Что находится в репозитории

- Разделы по архитектуре платформы: агенты, runtime‑слой (LM Studio / vLLM / LiteLLM), память, поиск (RAG), ingestion и эксплуатация.  
- Легенды и пояснения к архитектурным схемам (контуры gateway, guardrails, MCP, data contracts).  
- Навигационные карты по документации, маршруты чтения, troubleshooting и audit‑разделы.  
- Справочные материалы в `docs/generated/` для ускоренной ориентации в системе.

Документация сохраняет **архитектурную точность** и фокусируется на контрактном описании слоёв и потоков, без листинга кода. Это handbook для архитекторов и инженеров, которые строят или развивают enterprise‑agent платформу.
