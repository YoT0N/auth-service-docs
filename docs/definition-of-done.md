# Definition of Done (Documentation) — Auth Service

**Версія:** v1.0  
**Останнє оновлення:** 2025  
**Автор:** Ільків Богдан

> Цей документ визначає обов'язкові критерії готовності для кожного типу змін у проєкті з точки зору документації. Зміна вважається завершеною лише за умови виконання **всіх** відповідних пунктів.

---

## 1. Типи змін та обов'язкові дії

### Таблиця DoD

| Тип зміни | Обов'язкові артефакти | Автоматизовані перевірки | Ручна перевірка |
|-----------|----------------------|--------------------------|-----------------|
| Додано новий API-ендпоінт | Оновити `openapi.yaml` (новий path + schemas) | OpenAPI validation (spectral) | Перевірити Swagger UI |
| Змінено request/response структуру | Оновити `openapi.yaml` (schemas) + приклади | OpenAPI validation | Перевірити приклади у Swagger |
| Breaking change в API | Нова мажорна версія в `openapi.yaml`, CHANGELOG | OpenAPI validation + version check | Code review, оголошення |
| Додано UI-компонент | Додати Story у Storybook (default + стани) | Storybook build (без помилок) | Перевірити рендеринг Stories |
| Змінено UI-компонент | Оновити відповідну Story | Storybook build | Перевірити всі стани компонента |
| Змінено функціональну вимогу | Оновити SSD.md, SDD.md (якщо потрібно) | Markdown lint, link check | Узгодження з командою |
| Нефункціональні зміни (рефакторинг) | Оновити onboarding.md (якщо змінилась структура) | Markdown lint | — |

---

## 2. Детальні критерії для кожного типу

### 2.1 Зміна / додавання API-ендпоінта

**Обов'язково:**
- [ ] `docs/openapi.yaml` містить новий/оновлений path з усіма HTTP-методами
- [ ] Описані всі можливі response коди (200/201/400/401/409 тощо)
- [ ] Схеми request та response визначені в `components/schemas`
- [ ] Додані приклади (`example` або `examples`) для request/response
- [ ] Swagger UI відображає новий ендпоінт коректно (локальна перевірка)

**Автоматичні перевірки у CI:**
```
✓ spectral lint docs/openapi.yaml       — валідація OpenAPI специфікації
✓ redoc-cli build docs/openapi.yaml     — успішна збірка Swagger UI
```

---

### 2.2 Breaking change в API

**Breaking change** — будь-яка зміна, що порушує зворотну сумісність:
- видалення ендпоінта або поля
- зміна типу поля
- зміна HTTP-методу або URL

**Обов'язково:**
- [ ] Версія в `openapi.yaml` (поле `info.version`) підвищена на MAJOR (`1.x` → `2.0`)
- [ ] `CHANGELOG.md` містить опис breaking change
- [ ] Git tag створено: `git tag v2.0.0`
- [ ] Стара версія документації збережена в архіві (`/v1.0/`)
- [ ] Повідомлення команди / споживачів API

**Автоматичні перевірки у CI:**
```
✓ spectral lint docs/openapi.yaml
✓ version tag відповідає openapi.yaml info.version
```

---

### 2.3 Додавання / зміна UI-компонента

**Обов'язково:**
- [ ] Новий компонент має Story у директорії `src/stories/`
- [ ] Story містить мінімум 2 стани: `Default` + альтернативний (`Disabled` / `Error` / `Loading`)
- [ ] `storybook build` завершується без помилок
- [ ] Компонент коректно відображається в Storybook (локальна перевірка)

**Автоматичні перевірки у CI:**
```
✓ npm run build-storybook               — збірка без помилок
```

---

### 2.4 Зміна функціональної вимоги

**Обов'язково:**
- [ ] `docs/architecture/SSD.md` оновлено (Single Source of Truth)
- [ ] Якщо змінилась архітектура — оновлено `docs/architecture/SDD.md`
- [ ] `docs/quality/traceability-matrix.md` синхронізовано з новими вимогами
- [ ] Markdown lint не виявляє помилок

**Автоматичні перевірки у CI:**
```
✓ markdownlint docs/**/*.md
✓ lychee docs/**/*.md                  — перевірка посилань
✓ mkdocs build                         — збірка загальної документації
```

---

### 2.5 Нефункціональні зміни (рефакторинг, інфраструктура)

**Обов'язково:**
- [ ] Якщо змінилась структура проєкту — оновлено `docs/developer/onboarding.md`
- [ ] Якщо змінилась інфраструктура — оновлено `docs/architecture/ISD.md`
- [ ] Markdown lint не виявляє помилок

---

## 3. Загальні критерії готовності (для всіх типів змін)

```
Definition of Done — загальний чеклист
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
□ Всі документаційні артефакти оновлені відповідно до типу зміни
□ CI pipeline пройшов без помилок (validate + build stages)
□ Pull Request отримав мінімум 1 approve від reviewer
□ PR description містить посилання на вимогу (FR-XX або задачу)
□ Документація не містить битих посилань (lychee check)
□ Markdown файли проходять lint без попереджень
```

---

## 4. Де перевіряється

| Перевірка | Де відбувається | Відповідальний |
|-----------|----------------|----------------|
| OpenAPI validation | GitHub Actions (CI) | Автоматично |
| Storybook build | GitHub Actions (CI) | Автоматично |
| MkDocs build | GitHub Actions (CI) | Автоматично |
| Markdown lint | GitHub Actions (CI) | Автоматично |
| Перегляд Swagger UI | Локально / Staging | Розробник |
| Code review | GitHub Pull Request | Reviewer |
| Версія документації | Вручну при release | Tech Lead |

---

## 5. Приклад застосування DoD

**Сценарій:** Додаємо новий ендпоінт `POST /api/auth/forgot-password`

```
1. Розробник додає ендпоінт у код
2. Оновлює docs/openapi.yaml — додає /api/auth/forgot-password path
3. Описує ForgotPasswordRequest schema у components/schemas
4. Вказує responses: 200, 400, 404
5. Локально запускає: npx spectral lint docs/openapi.yaml ✓
6. Локально відкриває Swagger UI — перевіряє відображення ✓
7. Створює PR з описом: "Implements FR-06 — Password Reset"
8. CI pipeline запускається: validate → build → (publish на staging)
9. Reviewer перевіряє PR та специфікацію
10. Merge у main → публікація на GitHub Pages ✓
```

**Зміна вважається завершеною** лише після успішного проходження всіх кроків.
