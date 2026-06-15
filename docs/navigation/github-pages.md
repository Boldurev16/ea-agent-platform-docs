# Публикация документации на GitHub Pages

## Предпосылки

- Документация в каталоге **`/docs`** репозитория.
- Entry point: **`docs/index.md`**.

## Шаги

1. Откройте репозиторий на GitHub → **Settings** → **Pages**.
2. **Build and deployment**
   - Source: **Deploy from a branch**
   - Branch: `main` (или ваша default branch)
   - Folder: **`/docs`**
3. Save. Через 1–3 минуты появится URL вида  
   `https://<username>.github.io/<repo>/`.

## Структура для читателей

| URL path (пример) | Страница |
|-------------------|----------|
| `/` | [index.md](../index.md) |
| `/overview/` | Executive + PRD |
| `/architecture/` | System architecture |
| `/navigation/troubleshooting-map` | Incident map |

## Jekyll

Файл [`_config.yml`](../_config.yml) задаёт title и theme для GitHub Pages Jekyll build.

Mermaid-диаграммы в `generated/` могут требовать просмотра на github.com или с theme с поддержкой Mermaid — для production рассмотрите MkDocs/Docusaurus (вне scope текущей задачи).

## Локальный preview (optional)

```bash
# Ruby + bundler, если используете Jekyll локально
cd docs
bundle exec jekyll serve
```

## Не публикуются через Pages

- `../plans/` — вне `/docs` (ссылки из docs ведут на GitHub source view)
- Application code, `scripts/` — только упоминания в тексте

[← Navigation](index.md) · [← Главная](../index.md)
