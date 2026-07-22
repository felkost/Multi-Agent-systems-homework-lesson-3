# Contributing

Цей проєкт — навчальний (домашнє завдання з Multi-Agent Systems), тож процес
свідомо легкий. Ці правила діють однаково для соло-роботи й для будь-кого, хто
приєднається пізніше.

## Налаштування середовища

```bash
git clone <repo>
cd MA_systems_hl3
python -m venv .venv
.venv\Scripts\Activate.ps1          # Windows PowerShell
pip install -r requirements.txt -r requirements-dev.txt
cp .env.example .env                # заповнити OPENAI_API_KEY і MODEL_NAME
```

Запуск: `python main.py`.

## Перед комітом

```bash
black .
flake8 .
mypy .
pytest
```

`black`/`flake8` мають бути чистими завжди. `mypy`/`pytest` можуть тимчасово
показувати відомі помилки, поки `tools.py`/`agent.py` не реалізовані повністю —
це очікувано на ранньому етапі (див. `CLAUDE.md`), а не привід ігнорувати
інструменти надалі.

**Анотації типів обов'язкові** для всіх нових функцій, методів і атрибутів
класів — параметри й повернення. Це enforced-правило (`mypy` налаштований з
`disallow_untyped_defs`/`disallow_incomplete_defs`), а не рекомендація: PR із
функцією без повної анотації не пройде `mypy .`.

## Гілки та коміти

- Гілки: `feat/...`, `fix/...`, `chore/...` — короткий опис через дефіс.
- Коміти: короткий опис у формі вже зробленого (`added web_search tool`,
  `fixed thread_id persistence`) — без суворого Conventional Commits, головне
  щоб history залишалась читабельною.

## Pull Request

- Використовуйте шаблон `.github/PULL_REQUEST_TEMPLATE.md`.
- CI (`.github/workflows/ci.yml`) ганяє `black --check`, `flake8`, `mypy`,
  `pytest` на кожен push/PR. `black`/`flake8` мають проходити завжди; `mypy`
  наразі позначений `continue-on-error` через незавершені `tools.py`-заглушки
  і буде посилений, коли реалізацію завершено.
- API-ключі та секрети — лише в `.env` (не комітиться). Ніколи не вставляйте
  їх у код, коміти чи опис PR.

## Ліцензія

Проєкт розповсюджується під [MIT License](LICENSE). Роблячи внесок, ви
погоджуєтесь, що він розповсюджується на тих самих умовах.
