# CI/CD Pipeline для документації — Auth Service

**Версія:** v1.0  
**Останнє оновлення:** 2025  
**Автор:** Ільків Богдан

---

## 1. Артефакти документації

У проєкті Auth Service автоматичній генерації підлягають такі артефакти:

| Артефакт | Інструмент | Результат |
|----------|-----------|-----------|
| API-документація | Swagger UI | Інтерактивний HTML-сайт |
| UI-документація | Storybook | Статичний сайт компонентів |
| Загальна документація | MkDocs Material | Статичний сайт (з ЛР-8) |
| OpenAPI-специфікація | Валідатор | Перевірений YAML-файл |

---

## 2. Тригери генерації

```
Push у будь-яку feature-гілку
  └── Запускається: валідація OpenAPI + збірка (без публікації)

Push або merge у develop
  └── Запускається: повна збірка + публікація на staging

Merge у main (Production release)
  └── Запускається: повна збірка + публікація на GitHub Pages (production)
  └── Створюється: нова версія документації (version tag)
```

---

## 3. Структура Pipeline

### 3.1 Діаграма pipeline

```
GitHub Push / PR Merge
        │
        ▼
┌────────────────────────────────────┐
│     STAGE 1: Validate              │
│  • Перевірка OpenAPI (spectral)    │
│  • Lint Markdown (markdownlint)    │
│  • Перевірка посилань (lychee)     │
└────────────────┬───────────────────┘
                 │ (success)
                 ▼
┌────────────────────────────────────┐
│     STAGE 2: Build                 │
│  • Swagger UI (redoc-cli build)    │
│  • Storybook (build-storybook)     │
│  • MkDocs (mkdocs build)           │
└────────────────┬───────────────────┘
                 │ (success)
                 ▼
┌────────────────────────────────────┐
│     STAGE 3: Publish               │
│  (тільки для main / develop)       │
│  • Deploy до GitHub Pages          │
│  • Оновлення версії документації   │
└────────────────────────────────────┘
```

### 3.2 Послідовність кроків (детально)

#### Stage 1 — Валідація

```yaml
# .github/workflows/docs.yml (фрагмент)
- name: Validate OpenAPI specification
  run: npx @stoplight/spectral-cli lint docs/openapi.yaml

- name: Lint Markdown files
  run: npx markdownlint-cli docs/**/*.md

- name: Check broken links
  uses: lycheeverse/lychee-action@v1
  with:
    args: docs/**/*.md
```

#### Stage 2 — Збірка

```yaml
- name: Build Swagger UI
  run: |
    npx redoc-cli build docs/openapi.yaml \
      --output site/api/index.html \
      --title "Auth Service API"

- name: Build Storybook
  run: npm run build-storybook -- --output-dir site/storybook

- name: Build MkDocs
  run: mkdocs build --site-dir site/docs
```

#### Stage 3 — Публікація

```yaml
- name: Publish to GitHub Pages
  if: github.ref == 'refs/heads/main'
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./site
```

---

## 4. Структура URL публікації

Документація публікується на **GitHub Pages**:

```
https://<org>.github.io/auth-service/
│
├── /                    ← MkDocs — загальна документація
├── /api/                ← Swagger UI — API reference
└── /storybook/          ← Storybook — UI компоненти
```

| Середовище | URL |
|-----------|-----|
| Production (main) | `https://ilkiv.github.io/auth-service/` |
| Staging (develop) | Netlify Preview або окрема GitHub Pages гілка |

---

## 5. Versioning документації

### Правила версіонування

Версія документації **відповідає версії продукту** (semver: `MAJOR.MINOR.PATCH`):

| Тип зміни | Версія | Приклад |
|-----------|--------|---------|
| Breaking change в API | MAJOR | `v1.0` → `v2.0` |
| Новий ендпоінт / функція | MINOR | `v1.0` → `v1.1` |
| Виправлення / уточнення | PATCH | `v1.0` → `v1.0.1` |

### Процес створення нової версії

```
1. Змінити версію в openapi.yaml (поле info.version)
2. Оновити CHANGELOG.md
3. Створити Git tag: git tag v1.1.0
4. Push tag → автоматично запускається pipeline з публікацією нової версії
```

### Зберігання старих версій

Архів попередніх версій доступний за URL:
```
https://<org>.github.io/auth-service/v1.0/
https://<org>.github.io/auth-service/v1.1/
https://<org>.github.io/auth-service/latest/   ← завжди остання версія
```

---

## 6. Повний GitHub Actions Workflow

```yaml
name: Documentation CI/CD

on:
  push:
    branches: [main, develop, 'feature/**']
  pull_request:
    branches: [main, develop]

jobs:
  validate:
    name: Validate Documentation
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Validate OpenAPI specification
        run: npx @stoplight/spectral-cli lint docs/openapi.yaml

      - name: Lint Markdown
        run: npx markdownlint-cli "docs/**/*.md"

      - name: Check links
        uses: lycheeverse/lychee-action@v1
        with:
          args: "docs/**/*.md --exclude-loopback"

  build:
    name: Build Documentation
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install Node dependencies
        run: npm ci

      - name: Install MkDocs
        run: pip install mkdocs-material

      - name: Build Swagger UI
        run: |
          npx redoc-cli build docs/openapi.yaml \
            --output site/api/index.html \
            --title "Auth Service API v1.0"

      - name: Build Storybook
        run: npm run build-storybook -- --output-dir site/storybook

      - name: Build MkDocs
        run: mkdocs build --site-dir site/docs

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: docs-site
          path: site/

  publish:
    name: Publish to GitHub Pages
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: docs-site
          path: site/

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
          commit_message: "docs: deploy ${{ github.sha }}"
```
