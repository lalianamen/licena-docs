# LICENA — база знаний проекта

Этот репозиторий — **единая база знаний проекта LICENA** (licena.us):
фактическая документация для разработчиков и независимого аудита ChatGPT,
собранная только из реальных данных основного репозитория
[`lalianamen/llicena`](https://github.com/lalianamen/llicena).

Правила ведения — в [`CLAUDE.md`](CLAUDE.md), включая аудиторский стандарт
документа (§9): только проверяемые факты, каждое утверждение со ссылкой на
источник в проекте, `UNKNOWN` для отсутствующих данных, обязательные разделы
«Source References» и «Verification Status», без выводов и оценок
(бизнес-аудит делает ChatGPT в `30_CTO_REPORT.md`).

## Документы

| Файл | Содержание | Статус |
|---|---|---|
| [`01_PROJECT_OVERVIEW.md`](01_PROJECT_OVERVIEW.md) | Что такое LICENA: продукт, статус, курсы, языки, объём контента | создан |
| [`02_ARCHITECTURE.md`](02_ARCHITECTURE.md) | Архитектура: статический фронтенд + Supabase + Stripe, модули и потоки | создан |
| [`03_REPO_STRUCTURE.md`](03_REPO_STRUCTURE.md) | Структура основного репозитория с назначением каталогов и файлов | создан |
| [`04_DATABASE.md`](04_DATABASE.md) | Postgres: таблицы, RLS, функции, триггеры, cron, storage | создан |
| [`05_API.md`](05_API.md) | Вся API-поверхность: PostgREST, RPC, Edge Functions, внешние API | создан |
| [`06_FUNCTIONS.md`](06_FUNCTIONS.md) | Устройство всех 8 Edge Functions, секреты (имена), деплой-режимы | создан |
| [`07_AUTH_ACCESS.md`](07_AUTH_ACCESS.md) | Аутентификация, пути выдачи доступа, анти-шеринг, роли ключей | создан |
| [`08_SECURITY.md`](08_SECURITY.md) | Механизмы защиты, задокументированные слабые места, статус security-todo | создан |
| [`09_PAYMENTS.md`](09_PAYMENTS.md) | Stripe: модель, поток покупки, жизненный цикл подписки, триал | создан |
| [`10_CONTENT_MODEL.md`](10_CONTENT_MODEL.md) | Форматы банков, SOP генерации, чекер, процесс платного контента | создан |
| [`11_I18N.md`](11_I18N.md) | Все слои локализации EN/ES/RU, выбор языка, staged `hy` | создан |
| [`12_SEO.md`](12_SEO.md) | seo.js, JSON-LD, language-in-PATH, sitemap/robots, GSC-механика | создан |
| [`13_UX.md`](13_UX.md) | Полный перечень функциональности (Features) по поверхностям | создан |
| [`14_ANALYTICS.md`](14_ANALYTICS.md) | First-party аналитика: биконы, каналы, ежедневный отчёт | создан |
| [`15_METRICS.md`](15_METRICS.md) | Только реальные числа: контент, размеры, GSC-экспорт | создан |
| [`16_PERFORMANCE.md`](16_PERFORMANCE.md) | Механизмы производительности и измеренные размеры; тестов нет | создан |
| [`17_TECH_DEBT.md`](17_TECH_DEBT.md) | Подтверждённый долг (синхронизации, устаревшие описания) и гипотезы | создан |
| [`18_DEPENDENCIES.md`](18_DEPENDENCIES.md) | Все зависимости с версиями, местами использования и источниками версий | создан |
| [`19_INFRASTRUCTURE.md`](19_INFRASTRUCTURE.md) | Pages, Supabase, Stripe, Actions, email, соцканалы; UNKNOWN-границы | создан |
| [`20_VERIFICATION.md`](20_VERIFICATION.md) | Чекеры verify.js, ручной чек-лист, ship-гейты, области без проверок | создан |
| [`21_AGENT_PROCESS.md`](21_AGENT_PROCESS.md) | Оркестратор + 5 лейнов, конвейеры (тикеты, cron-аудит, автопост) | создан |
| [`26_CHANGELOG.md`](26_CHANGELOG.md) | История продукта (задокументированная) + журнал базы знаний | создан |
| [`28_AI_CONTEXT.md`](28_AI_CONTEXT.md) | Сводный контекст для AI-сессий: инварианты, процессы, где что лежит | создан |

## Аудит

| Файл | Содержание |
|---|---|
| [`audit/INDEX.md`](audit/INDEX.md) | Реестр всех документов: статус заполнения, дата последней проверки, уровень достоверности |
| [`audit/COVERAGE.md`](audit/COVERAGE.md) | Карта покрытия: какие части основного проекта задокументированы, какие нет |

Остальные файлы аудита (`22_MARKETING_STATE.md`, `23_SMM_STATE.md`,
`24_RISKS.md`, `25_DECISIONS.md`, `27_ROADMAP.md`,
`29_BUSINESS_FRAMEWORK.md`, `30_CTO_REPORT.md`, `RELEASE_SUMMARY.md`,
`rfc/`) перечислены в [`CLAUDE.md`](CLAUDE.md) и
[`audit/INDEX.md`](audit/INDEX.md); они ещё не созданы — пустые шаблоны не
создаются намеренно.

## Source References

- [`CLAUDE.md`](CLAUDE.md) этого репозитория (канонический список документов);
- фактическое содержимое репозитория licena-docs на дату сверки.

## Verification Status

**Verified** — README описывает только состав этого репозитория.

Последняя сверка: 2026-08-05
