# RAG-ассистент

RAG (Retrieval-Augmented Generation) ассистент, реализованный в двух вариантах: на базе **OpenAI API** и на базе **GigaChat API** (Сбер). Ассистент отвечает на вопросы по теме машинного обучения, NLP и смежных областей, используя векторный поиск по базе знаний.

## Архитектура

```
┌─────────────────────────────────────────────┐
│           app.py (CLI — точка входа)         │
├─────────────────────────────────────────────┤
│              rag_pipeline.py                 │
│         (оркестратор RAG-пайплайна)          │
├──────────────────┬──────────────────────────┤
│  vector_store.py  │        cache.py          │
│  (ChromaDB +      │    (SQLite-кеш          │
│   embeddings)      │     вопрос-ответ)       │
└──────────────────┴──────────────────────────┘
```

### Поток обработки запроса

1. Кеш (SQLite) — проверка, был ли уже такой вопрос
2. Векторный поиск (ChromaDB) — поиск top-3 релевантных документов
3. Формирование промпта — контекст + вопрос
4. Генерация ответа — LLM (OpenAI / GigaChat)
5. Сохранение в кеш

## Состав проекта

```
RAG-ассистент/
├── .env                                # API-ключи
├── requirements.txt                    # зависимости
│
├── assistant_api/                      # версия на OpenAI
│   ├── app.py                          # CLI-интерфейс
│   ├── rag_pipeline.py                 # RAG-пайплайн
│   ├── vector_store.py                 # ChromaDB + эмбеддинги OpenAI
│   ├── cache.py                        # SQLite-кеш
│   ├── evaluate_ragas.py               # оценка через RAGAS
│   └── data/docs.txt                   # база знаний (30 документов)
│
└── assistant_giga/                     # версия на GigaChat
    ├── app.py                          # CLI-интерфейс
    ├── rag_pipeline.py                 # RAG-пайплайн
    ├── vector_store.py                 # ChromaDB + эмбеддинги GigaChat
    ├── cache.py                        # SQLite-кеш
    ├── gigachat_client.py              # HTTP-клиент GigaChat API
    └── data/docs.txt                   # база знаний (31 документ)
```

## Установка и запуск

### 1. Настройка виртуального окружения

```powershell
python -m venv venv
.\venv\Scripts\activate
```

### 2. Установка зависимостей

```powershell
pip install -r requirements.txt
```

### 3. Настройка `.env`

Заполните файл `.env` в корне проекта:

```env
OPENAI_API_KEY=sk-...                    # для assistant_api
OPENAI_BASE_URL=https://openrouter.ai/api/v1  # для OpenRouter
GIGACHAT_RQUID=...                       # для assistant_giga
GIGACHAT_AUTH_KEY=...                    # для assistant_giga
```

### 4. Запуск

**OpenAI версия:**

```powershell
cd assistant_api
.\..\venv\Scripts\activate
python app.py
```

**GigaChat версия:**

```powershell
cd assistant_giga
.\..\venv\Scripts\activate
python app.py
```

### Команды в CLI

| Команда | Описание |
|---------|----------|
| `любой вопрос` | Задать вопрос ассистенту |
| `stats` | Показать статистику (размер коллекции, записи кеша, модель) |
| `clear` | Очистить кеш |
| `exit` / `quit` / `q` | Выйти |

## Технологии

| Компонент | OpenAI версия | GigaChat версия |
|-----------|---------------|-----------------|
| LLM | `gpt-4o-mini` | `GigaChat` |
| Эмбеддинги | `text-embedding-3-small` | `Embeddings` (GigaChat) |
| Векторная БД | ChromaDB (cosine) | ChromaDB (cosine) |
| Кеш | SQLite | SQLite |
| Оценка | RAGAS (faithfulness, context precision) | — |

## Оценка качества (только OpenAI версия)

```powershell
cd assistant_api
python evaluate_ragas.py
```

Используются метрики RAGAS: **Faithfulness** (точность по контексту) и **Context Precision** (релевантность контекста).

## База знаний

Файл `data/docs.txt` содержит документы на русском языке по темам:
- Машинное обучение, нейронные сети, NLP
- Трансформеры, эмбеддинги
- RAG, векторные базы данных
- Prompt engineering, fine-tuning
- Оценка LLM, кеширование, chunking
- Температура, token limits, semantic search
- GigaChat (только в `assistant_giga`)
