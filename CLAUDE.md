# CLAUDE.md

Інструкції для Claude Code (та інших агентів) під час роботи в цьому репозиторії.

## Що це за проєкт

Термінальний **Research Agent**: користувач ставить дослідницьке питання, агент сам
планує пошук, викликає інструменти (`web_search`, `read_url`, `write_report`) і
зберігає структурований Markdown-звіт у `output/`.

**Це один агент, не мультиагентна система.** Назва репозиторію згадує
"Multi-Agent Systems" (курс, у межах якого виконується завдання), але сама
задача — single-agent ReAct loop через LangChain `create_agent`. Не додавайте
supervisor/subagents, роутери чи інші мультиагентні патерни — вони не випливають
із вимог і будуть надмірним ускладненням.

## Архітектура

| Файл | Роль |
|---|---|
| `main.py` | Entry point. Інтерактивний REPL: читає питання з терміналу, стрімить відповідь агента. Має тримати **сталий `thread_id`** в межах усієї сесії — це єдиний спосіб, яким `MemorySaver`/checkpointer памʼятає попередні повідомлення. |
| `agent.py` | Створення агента: LLM (`ChatOpenAI` через `langchain-openai`), три tools, checkpointer (`InMemorySaver`/`MemorySaver`), middleware для лімітів кроків. |
| `tools.py` | Реалізації `web_search`, `read_url`, `write_report` як LangChain `@tool` — з описами, які бачить модель. |
| `config.py` | `Settings` (Pydantic Settings, читає `.env`) + `SYSTEM_PROMPT`. Усі константи й промпти — тут, не хардкодити в `agent.py`/`tools.py`. |

## Команди розробки

```bash
python -m venv .venv
.venv\Scripts\Activate.ps1          # Windows PowerShell
pip install -r requirements.txt -r requirements-dev.txt
cp .env.example .env                # і заповнити ключ
python main.py
```

Лінт і тести (перед комітом — див. `CONTRIBUTING.md`):

```bash
black --check .
flake8 .
mypy .
pytest
```

## Інваріанти, які не видно з коду

- **Памʼять працює лише якщо `thread_id` сталий** для всієї сесії `main.py`
  (`config = {"configurable": {"thread_id": thread_id}}`, той самий `config` —
  у кожен виклик агента). Новий `thread_id` на кожен запит = агент "забуває"
  попередні повідомлення, хоч checkpointer підключений.
- **Ліміт кроків задається через middleware** (`ToolCallLimitMiddleware` /
  `ModelCallLimitMiddleware`), а не через `Settings.max_iterations` — це поле
  саме по собі нічого не обмежує, поки не підключене до агента.
- **`output/` за замовчуванням ігнорується git, але Markdown-звіти зберігаються і комітяться** (`.gitignore`: `output/*`, `!output/.gitkeep`, `!output/*.md`) — `.gitkeep` лишається для гарантії існування папки, а звіти у форматі `*.md` у `output/` повинні бути закомічені разом з PR (наприклад `output/report.md`).

- **Рекомендація щодо звітів:** зберігайте кінцевий структурований звіт у `output/` як Markdown і додайте короткий опис у PR; якщо звіт виводиться тимчасово під час сесії, виконуйте окремий крок "згенерувати → зберегти" щоб гарантувати запис файлу.
- **mypy `.` виявляє реальні помилки на заглушках `tools.py` й `agent.py`**
  (`Missing return statement [empty-body]` у `tools.py`; `"EllipsisType" has no
  attribute "stream"` у `main.py`, бо `agent.py` поки що `agent = ...`) — це
  очікувано, доки код не реалізований, а не збій конфігурації.

## Стиль коду

- **Анотації типів обов'язкові** для всіх функцій, методів і атрибутів класів
  (параметри й повернення) — без винятків для "простих" функцій. Це не лише
  побажання: `mypy` налаштований з `disallow_untyped_defs` і
  `disallow_incomplete_defs` (`pyproject.toml`), тож функція без повної
  анотації — це помилка `mypy .`, а не стиль на розсуд автора.

## Заборонено

- Комітити `.env` або будь-які API-ключі.
- Хардкодити порядок викликів tools у `main.py`/`agent.py` — модель сама вирішує
  послідовність.
- Додавати функції/методи без повних анотацій типів (див. "Стиль коду" вище).

## Тести

`tests/` наразі не існує — його свідомо не створювали, доки `tools.py`/`agent.py`
залишаються заглушками (тести на порожні функції не мають сенсу). Додавайте
`tests/test_tools.py`, `tests/test_config.py`, `tests/test_memory.py` разом із
реалізацією відповідного коду, з мокнутими мережевими викликами.
