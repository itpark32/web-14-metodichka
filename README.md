# Методичка 14 потока по веб-разработке

Полный маршрут повторения курса: HTML, CSS, JavaScript, React, TypeScript, тестирование, архитектура, PHP, MySQL и Laravel. Отдельный раздел готовит к техническому интервью по всем темам программы.

## Локальный запуск

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-docs.txt
mkdocs serve
```

Откройте `http://127.0.0.1:8000`.

## Проверка перед публикацией

```bash
mkdocs build --strict
```

Сайт автоматически публикуется в GitHub Pages после отправки изменений в ветку `main`.
