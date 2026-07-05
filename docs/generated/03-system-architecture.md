# 03 — System architecture

Этот раздел объясняет платформу как набор связанных контуров управления: диалог, агентное рассуждение, поиск по знаниям, загрузка документов, память и эксплуатационная готовность.

## 1. Контекст системы

```mermaid
flowchart TB
  subgraph actors["Пользователи"]
    businessUser["Бизнес-пользователь"]
    architect["Enterprise architect"]
    operator["Оператор базы знаний"]
    support["Эксплуатация"]
  end

  subgraph platform["ea-agent-platform"]
    api["API и чат"]
    agent["Agent runtime"]
    tools["Инструменты агента"]
    retrieval["Retrieval"]
    ingestion["Ingestion worker"]
    memory["Память сессии"]
  end

  subgraph infra["Инфраструктурные зависимости"]
    llm["LLM runtime"]
    embeddings["Embedding runtime"]
    vectorDb["Vector store"]
    objectStore["Object storage"]
    database["Postgres"]
  end

  businessUser --> api
  architect --> api
  operator --> ingestion
  support --> api
  api --> agent
  agent --> llm
  agent --> tools
  tools --> retrieval
  retrieval --> embeddings
  retrieval --> vectorDb
  ingestion --> embeddings
  ingestion --> vectorDb
  ingestion --> objectStore
  agent --> memory
  memory --> database
  ingestion --> database
```

## 2. Как читать схему

| Контур | Управленческий смысл |
|--------|----------------------|
| Чат | Единая точка доступа к архитектурной функции. |
| Agent runtime | Контур рассуждения и выбора действия. |
| Tools | Разрешенные способы взаимодействия агента с внешними знаниями и сервисами. |
| Retrieval | Проверяемое извлечение фрагментов из базы знаний. |
| Ingestion | Производственный путь включения новых документов в knowledge base. |
| Memory | Контекст текущего диалога, а не источник истины. |
| Operations | Контроль готовности и деградации зависимостей. |

## 3. Основной поток ответа

```mermaid
sequenceDiagram
  participant User as Пользователь
  participant API as Чат API
  participant Agent as Agent runtime
  participant LLM as LLM
  participant Tool as search_kb
  participant Retrieval as Retrieval
  participant KB as База знаний

  User->>API: Вопрос
  API->>Agent: query + session_id
  Agent->>LLM: Контекст и инструкции
  LLM-->>Agent: Шаг tool или final
  alt Нужен источник
    Agent->>Tool: Поисковый запрос
    Tool->>Retrieval: Семантический поиск
    Retrieval->>KB: Поиск фрагментов
    KB-->>Retrieval: Релевантные фрагменты
    Retrieval-->>Tool: Ответ с citations
    Tool-->>Agent: Tool result
    Agent->>LLM: Контекст + tool result
  end
  Agent-->>API: События SSE
  API-->>User: Ответ, источники, trace
```

Главное архитектурное решение: RAG вызывается **только через инструмент**. Это сохраняет агенту возможность отвечать быстро на общие вопросы и обращаться к базе знаний только тогда, когда требуется обоснование.

## 4. Поток включения документов

```mermaid
flowchart LR
  files["Документы"] --> parse["Разбор формата"]
  parse --> normalize["Нормализация текста"]
  normalize --> chunks["Фрагменты"]
  chunks --> embed["Embeddings"]
  embed --> index["Индексирование"]
  index --> kb["База знаний"]
  files --> storage["Object storage"]
```

Ingestion — это промышленная дисциплина ведения знаний. Если документы не обработаны, агент не сможет использовать их как надежный источник, даже если они “где-то лежат”.

## 5. Слои реализации

| Слой | Назначение | Примеры модулей |
|------|------------|-----------------|
| API | HTTP, SSE, UI shell | `app/api/*` |
| Agent runtime | Tool loop и streaming | `orchestration/*` |
| Tools | Реестр разрешенных действий | `tools/*` |
| Retrieval | Поиск и citations | `retrieval/*` |
| Memory | История диалога | `memory/*` |
| Ingestion | Документы → база знаний | `ingestion/*` |
| LLM / embeddings | Модель ответа и модель смысла | `llm/*`, `embeddings/*` |
| Storage / DB | Jobs, sessions, object storage | `storage/*` |
| Core | Контракты, readiness, protocols | `core/*` |

## 6. Эксплуатационная модель

Архитектура различает:

- **Liveness** — приложение отвечает как процесс.
- **Readiness** — весь контур готов дать качественный ответ: доступны LLM/embeddings, vector store, база данных и storage.

Это важно для эксплуатации: “сервис жив” не означает “агент может дать ответ с источниками”.

## 7. Типовые деградации

| Деградация | Что увидит пользователь | Что проверяет эксплуатация |
|------------|-------------------------|----------------------------|
| LLM недоступна | Ответ не формируется | Health LLM |
| Embeddings недоступны | Поиск по базе знаний деградирует | Embedding runtime |
| Vector store недоступен | Нет релевантных источников | Readiness dependency |
| Postgres недоступен | Проблемы с jobs или session memory | Jobs и memory store |
| Плохой corpus | Ответы слабые, хотя сервис “здоров” | Качество ingestion и документов |

## 8. Достоверность раздела

Материал описывает санитизированную архитектурную картину. Имена модулей оставлены как учебные ориентиры, но без исходного кода, секретов и приватных конфигураций.

Связанные разделы: [Agent runtime](04-agent-runtime.md), [search_kb](06-tools-search_kb.md), [similarity_search](07-retrieval-similarity_search.md).
