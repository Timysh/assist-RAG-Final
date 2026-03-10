# 🧠 RAG-ассистент: ответы на вопросы по вашим документам

Консольные приложения с **Retrieval-Augmented Generation (RAG)** — вы задаёте вопрос, система находит релевантные фрагменты в ваших документах и генерирует ответ на их основе. Поддерживаются два режима: **OpenAI API** и **GigaChat (Сбер)**.

---

## 📌 Для чего нужен проект

- **Вопросы по своей базе знаний** — загружаете текстовые документы (`data/docs.txt`), система индексирует их в векторной БД и отвечает только по этому контексту.
- **Два бэкенда на выбор:**
  - **assistant_api** — OpenAI (GPT-4o-mini), эмбеддинги и генерация через API.
  - **assistant_giga** — GigaChat от Сбера (авторизация по ключу и RQUID).
- **Кеширование** — повторные запросы с тем же вопросом берутся из SQLite-кеша, без повторного вызова LLM.
- **Оценка качества (только API)** — скрипт `evaluate_ragas.py` считает метрики RAGAS (faithfulness, context precision) для проверки качества RAG.

Итог: один проект — два варианта RAG-ассистента (OpenAI или GigaChat) с векторным поиском, кешем и опциональной оценкой.

---

## 🏗 Структура проекта

```
PEr08mod5 Final Project/
├── assistant_api/           # RAG на OpenAI API
│   ├── app.py               # Точка входа (консоль)
│   ├── rag_pipeline.py      # Pipeline: кеш → поиск → LLM → кеш
│   ├── vector_store.py      # ChromaDB + эмбеддинги OpenAI
│   ├── cache.py             # SQLite-кеш вопрос–ответ
│   ├── evaluate_ragas.py    # Оценка RAG через RAGAS
│   └── data/
│       └── docs.txt         # Ваши документы для индексации
├── assistant_giga/          # RAG на GigaChat
│   ├── app.py               # Точка входа (консоль)
│   ├── rag_pipeline.py      # Pipeline для GigaChat
│   ├── vector_store.py      # ChromaDB + эмбеддинги (GigaChat/fallback)
│   ├── gigachat_client.py   # Клиент GigaChat API
│   ├── cache.py             # SQLite-кеш
│   └── data/
│       └── docs.txt         # Документы для индексации
├── requirements.txt         # Зависимости Python
├── .env                     # Ключи (не в репозитории)
└── README.md                # Этот файл
```

---

## ⚙️ Требования

- **Python 3.11** (рекомендуется)
- Файл **`.env`** в корне проекта с ключами (см. ниже)

---

## 🚀 Установка и запуск

### 1. Клонирование и окружение

```bash
cd "PEr08mod5 Final Project"
python -m venv venv
# Windows:
venv\Scripts\activate
# Linux/macOS:
# source venv/bin/activate
pip install -r requirements.txt
```

### 2. Переменные окружения

Создайте в корне проекта файл **`.env`** (он в `.gitignore`).

**Для assistant_api (OpenAI):**

```env
OPENAI_API_KEY=sk-...
```

**Для assistant_giga (GigaChat):**

```env
GIGACHAT_AUTH_KEY=ваш-auth-key
GIGACHAT_RQUID=ваш-rquid
```

> GigaChat: ключи получают в [кабинете разработчика Сбера](https://developers.sber.ru/).  
> Для эмбеддингов в GigaChat может использоваться fallback; при необходимости в `vector_store` можно переключить на OpenAI (если добавить `OPENAI_API_KEY`).

### 3. Данные для RAG

Положите текст для индексации в:

- **assistant_api:** `assistant_api/data/docs.txt`
- **assistant_giga:** `assistant_giga/data/docs.txt`

Формат: обычный текст, абзацы через пустую строку. При первом запуске коллекция ChromaDB создаётся и заполняется автоматически.

### 4. Запуск

**Режим OpenAI (API):**

```bash
cd assistant_api
python app.py
```

**Режим GigaChat:**

```bash
cd assistant_giga
python app.py
```

В консоли появится приветствие и приглашение ввести вопрос.

---

## 📖 Как пользоваться

1. **Ввод вопроса** — введите текст и нажмите Enter. Система проверит кеш, при необходимости выполнит поиск по документам и вызов LLM, выведет ответ и источник (кеш или модель).
2. **Команды в консоли:**
   - `stats` — статистика: коллекция ChromaDB, число записей в кеше, модель.
   - `clear` — очистка кеша (с подтверждением).
   - `exit` или `quit` — выход.

Повторный тот же вопрос выдаётся из кеша без повторного запроса к API.

---

## 📊 Оценка качества RAG (только assistant_api)

Для проверки качества RAG по метрикам RAGAS (faithfulness, context precision):

```bash
cd assistant_api
python evaluate_ragas.py
```

Скрипт использует тестовые вопросы из кода, прогоняет их через RAG (без кеша) и выводит метрики. Нужен `OPENAI_API_KEY` (используется и для RAG, и для RAGAS).

---

## 🛠 Технологии

| Компонент        | Назначение                          |
|------------------|-------------------------------------|
| **ChromaDB**     | Векторное хранилище, поиск по смыслу |
| **OpenAI API**   | Эмбеддинги и GPT-4o-mini (assistant_api) |
| **GigaChat API** | Модель и опционально эмбеддинги (assistant_giga) |
| **SQLite**       | Кеш пар вопрос–ответ                |
| **RAGAS**        | Оценка качества RAG (assistant_api) |
| **python-dotenv**| Загрузка `.env`                     |

---

## 📄 Лицензия и контекст

Для продакшена стоит вынести секреты в безопасное хранилище и при необходимости донастроить разбиение документов и параметры поиска (top_k, chunk_size и т.д.).

Если что-то по README или запуску непонятно — можно уточнить по конкретной папке (`assistant_api` или `assistant_giga`) или по шагу установки.
