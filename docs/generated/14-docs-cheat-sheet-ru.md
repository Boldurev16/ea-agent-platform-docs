# 14 — Шпаргалка по учебному курсу

Эта шпаргалка помогает быстро вспомнить, где искать объяснение нужной темы.

## 1. Если нужно объяснить систему руководителю

Читайте:

1. [01-executive-overview.md](01-executive-overview.md)
2. [02-prd.md](02-prd.md)
3. [03-system-architecture.md](03-system-architecture.md)

Главная формулировка: платформа создает управляемый агентный контур, где ответ связан с корпоративными источниками и может быть проверен.

## 2. Если нужно понять runtime

Читайте:

1. [04-agent-runtime.md](04-agent-runtime.md)
2. [06-tools-search_kb.md](06-tools-search_kb.md)
3. [11-api-and-integration-points.md](11-api-and-integration-points.md)

Ключевая мысль: LLM не просто генерирует текст; runtime заставляет ее работать в структурированном цикле `tool` или `final`.

## 3. Если проблема в качестве источников

Читайте:

1. [07-retrieval-similarity_search.md](07-retrieval-similarity_search.md)
2. [09-ingestion-pipeline.md](09-ingestion-pipeline.md)
3. [13-open-questions.md](13-open-questions.md)

Ключевая мысль: качество ответа зависит от качества корпуса, chunking, embeddings и threshold.

## 4. Если проблема в эксплуатации

Читайте:

1. [12-ops-observability-and-risks.md](12-ops-observability-and-risks.md)
2. [08-ingestion-worker.md](08-ingestion-worker.md)
3. [../navigation/troubleshooting-map.md](../navigation/troubleshooting-map.md)

Ключевая мысль: “сервис жив” не означает “сервис готов дать ответ с источниками”.

## 5. Термины, которые нужно знать

| Термин | Коротко |
|--------|---------|
| RAG | Ответ с опорой на найденные знания. |
| `search_kb` | Инструмент поиска по корпоративной базе знаний. |
| Citation | Проверяемая ссылка на источник. |
| Tool loop | Цикл, в котором агент может вызвать инструмент и продолжить рассуждение. |
| Session memory | Контекст диалога, не источник истины. |
| Ingestion | Включение документов в базу знаний. |
| Readiness | Готовность всего контура, а не только процесса приложения. |

Полный словарь: [../navigation/glossary.md](../navigation/glossary.md).

## 6. Что не надо путать

| Не путать | Почему |
|-----------|--------|
| UI history и server memory | Первое нужно для экрана, второе — для prompt context. |
| Liveness и readiness | Первое про процесс, второе про готовность зависимостей. |
| LLM и база знаний | LLM рассуждает, база знаний дает источники. |
| Ingestion и retrieval | Ingestion готовит знания, retrieval ищет по ним. |
| MVP и product shell | Backend готов, но self-service UI еще развивается. |

## 7. Рекомендуемый первый час

1. [../index.md](../index.md)
2. [../navigation/cheat-sheet.md](../navigation/cheat-sheet.md)
3. [01-executive-overview.md](01-executive-overview.md)
4. [03-system-architecture.md](03-system-architecture.md)
5. [04-agent-runtime.md](04-agent-runtime.md)
