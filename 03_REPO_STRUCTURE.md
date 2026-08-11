# 03 — Структура основного репозитория (`lalianamen/llicena`)

Последняя сверка: 2026-08-05 (полная) · 2026-08-10 (точечная: HVAC-калькулятор)
Состояние: ветка `main`, 357 файлов (без `.git`). Ниже — полная структура с
назначением; роли взяты из header-комментариев самих файлов и `CLAUDE.md`
основного репо.

## Корень

| Путь | Назначение |
|---|---|
| `index.html` | Лендинг + вход/регистрация (Supabase Auth) |
| `app.html` | Кабинет (My courses / Catalog / Account), noindex |
| `course.html` | Плеер курса, noindex |
| `about/index.html`, `privacy.html`, `terms.html` | Публичные страницы |
| `CNAME` | Домен GitHub Pages: `licena.us` |
| `.nojekyll` | Отключение Jekyll-обработки на Pages |
| `robots.txt`, `sitemap.xml` (126 URL) | Краулинг; sitemap ведёт marketing-агент |
| `manifest.json`, `sw.js` | PWA: манифест + service worker (network-first, кэш `licena-v1`) |
| `favicon.svg/png`, `icon-192/512.png`, `apple-touch-icon.png`, `og-image.png` | Иконки и OG-картинка |
| `img/course-card-*.png`, `img/course-full-*.png` | Скриншоты продукта (EN/ES/RU) |
| `googlec4665304f2eceeb0.html` | Верификация Google Search Console |
| `README.md` | Описание проекта (частично устарело — см. `28_AI_CONTEXT.md`) |
| `CLAUDE.md` | Правила работы над кодом: Karpathy-гайдлайны, agent lanes, инварианты банков |
| `.gitignore` | Исключает `.env*`, `node_modules/`, генерируемые `supabase/sql/seed-*.sql` и `bank_questions.csv` |

## `js/` — весь клиентский код (24 модуля + данные)

| Путь | Назначение |
|---|---|
| `js/supabase-client.js` | Клиент Supabase (URL + publishable key, implicit flow) |
| `js/app.js`, `js/app-cabinet.js`, `js/app-course.js` | Логика трёх страниц приложения |
| `js/catalog.js`, `js/catalog/ca.js`, `js/catalog/az.js` | Реестр штатов + каталоги курсов Калифорнии и Аризоны |
| `js/i18n.js`, `js/i18n-app.js` | Переводы EN/ES/RU (лендинг / кабинет+плеер) |
| `js/paths.js` | Данные «path wizard» (лицензия → экзамены) |
| `js/seo.js` | Инжектор SEO-`<head>` (лейн marketing) |
| `js/pwa.js`, `js/devices.js`, `js/onboarding.js` | SW-регистрация; анти-шеринг клиент; туры |
| `js/stats.js`, `js/pageview.js` | First-party аналитика → `page_views` |
| `js/honest-chances.js`, `js/course-blocks.js`, `js/course-ref.js` | Контент-панели курса (лейн content) |
| `js/bank-updates.js` / `js/bank-updates-render.js` | Журнал обновлений банков / его рендер |
| `js/resources.js` / `js/resources-render.js` | Официальные ссылки (лейн resources) / рендер (лейн design) |
| `js/reviews.js`, `js/support.js` | Отзывы на лендинге; виджет AI-саппорта |
| `js/sample-quiz.js` | Движок публичных sample-квизов для `/practice/*` |
| `js/questions/` | **Статические банки (только бесплатные)**: 22 банка × (EN + `.es.js` + `.ru.js`) = 2 850 EN-вопросов. Платные банки удалены из `main` 2026-07-29 — их исходники на ветке `content-banks-src` (37 банков, 10 350 EN-вопросов), сайт отдаёт их из Supabase `bank_questions` |
| `js/samples/` | 20 публичных sample-наборов для SEO-страниц (отдельные от банков) |
| `js/guides/` | Контент курсов-гайдов: `contractor-business.js`, `cslb-license.js`, `cslb-license-intro.js` |
| `js/vendor/supabase-js-2.110.0.js` | Единственный vendor |

## `css/`

`styles.css` (база + токены + auth, 582 строки), `cabinet.css` (664),
`course.css` (514), `exam.css` (282).

Дополнение 2026-08-10 (влито в `main` 2026-08-10 (fast-forward `ed30701`→`32f7201`)): + `tools.css` (61 строка — страницы `/tools/`,
включая embed-режим); `course.css` дополнен стилями `.tool-card`.

Дополнение 2026-08-11 (`1e61960`, переписано в `04a2d99` при порте макета):
+ `landing.css` (229 строк) — тёмная тема navy+gold ТОЛЬКО для `index.html`
(переопределение токенов после `styles.css` + стили секций макета,
full-bleed банды, `body{overflow-x:clip}`; auth-модал и саппорт
пере-закреплены светлыми). Парный JS: `js/landing-extras.js` (155 строк) —
калькулятор макета (ползунки + чипы-пресеты) и живой вопрос C-10 с языковым
пиллом (данные — `js/samples/c10-exam.js`). В `img/` добавлено
`hero-contractor.jpg` (1200×1408, 143 KB — hero-фото из Lovable-образца
`lalianamen/licena-insights`, src/assets).

## SEO-страницы (статические, EN + зеркала `es/`, `ru/`)

| Путь | Что это |
|---|---|
| `practice/<exam>/index.html` — 20 шт. | Per-exam страницы с sample-квизом (c-10, c-20, c-36, cslb-law-and-business, epa-608, az-sre-practice, …) |
| `guides/<topic>/index.html` — 19 шт. | Гайды: лицензии CA (B, B-2, C-7…C-46), Arizona (license, exams, reciprocity), EPA 608, NICET ×2, OSHA 10/30, backflow |
| `es/…`, `ru/…` | Полные испанское и русское зеркала `about` + `practice` + `guides` |
| `tools/<slug>/index.html` — 10 шт. (+ зеркала `es/`, `ru/`) | Учебные калькуляторы (course-gated, noindex, вне sitemap): hvac-sizing (логика `js/hvac-calc.js`), hvac-design (Manual S/D), electrical, plumbing, roofing, concrete, painting, solar, fire, pricing (общая логика `js/trade-calcs.js`); добавлены 2026-08-10/11 (`32f7201`, `4234bff`, `6530354`) |

## `data/`

`questions.example.json` — образец схемы вопроса.

## `docs/` — внутренние документы (закрыты в `robots.txt`)

| Путь | Назначение |
|---|---|
| `docs/access-control.md` | Модель анти-шеринга устройств |
| `docs/security-todo.md` | Открытые security-задачи |
| `docs/schema.sql`, `docs/migration-2026-06-22-course-status.sql` | Базовая схема (profiles, devices, user_courses) и миграция |
| `docs/content/bank-playbook.md` | **Обязательный SOP генерации банков** (blueprint → fact ledger → батчи+чекер → переводы → ship-гейты) |
| `docs/content/*-blueprint.md`, `*-ledger.md` | Блюпринты и фактовый леджер конкретных банков (B, C-27) |
| `docs/content-audit/queue.json` | Очередь автоматической контент-сверки банков по официальным источникам (cron; статусы pending/done) |
| `docs/email/confirm-signup.html` | Шаблон письма подтверждения |
| `docs/marketing/` | Лейн marketing: `plan-2026-H2.md`, `growth-narrow-first-2026-07.md`, `seo-audit-2026-07-19.md`, `seo-backlog.md`, `decisions-log.md`, `gsc-readout-2026-08.md`, `spec-per-exam-pages.md`, `flyer/` (HTML+PDF флаеры 4×6/A6, EN-ES) |
| `docs/smm/` | Лейн smm: пост-паки по датам, `calendar.md`, лончкиты Instagram/TikTok, `queue/` и `queue-fb/` (txt-файлы → автопост через GitHub Actions), расписания |

## `supabase/` — бэкенд как код

| Путь | Назначение |
|---|---|
| `functions/assistant/` | AI-саппорт (Claude `claude-sonnet-4-6`, web search) |
| `functions/stripe-checkout/` | Checkout-сессия $20/мес за курс (14 платных курсов) |
| `functions/stripe-webhook/` | Единственный источник выдачи доступа (`user_courses`) |
| `functions/stripe-portal/` | Портал управления подпиской |
| `functions/ticket-issue/`, `ticket-email/`, `ticket-status/` | Тикеты: GitHub issue, письма, статус |
| `functions/daily-stats/` | Ежедневная статистика |
| `devices_anti_sharing.sql` | `register_device()`: 5 устройств, swap ≤1/30 дней |
| `reviews.sql`, `support_tickets.sql`, `support_uploads_storage.sql`, `ticket_*_trigger.sql` | Отзывы, тикеты, Storage, триггеры |
| `sql/bank-questions-schema.sql` | Таблица `bank_questions` (платные банки, RLS) |
| `sql/subscriptions-schema.sql`, `sql/stripe-payments.sql` | Подписки и платежи |
| `sql/trial-3day.sql` | 3-дневный триал (`course_trials`, `start_trial`) |
| `sql/tester-account.sql`, `sql/extend-tester-trials-oct31.sql` | Тестерские аккаунты (`profiles.is_tester`) |
| `sql/restore-beta-policies-until-aug1.sql` | Бета-политики «бесплатно до 31 июля» |
| `sql/page-views.sql`, `sql/report-kpi.sql`, `sql/cron-daily-stats.sql` | Аналитика: `page_views`, `social_stats`, cron |
| `sql/reviews-translations.sql`, `sql/backfill-business-guide.sql` | Переводы отзывов; бэкфилл гайда |
| `email-templates/confirm-signup.html`, `reset-password.html` | Шаблоны писем Supabase Auth |

## `scripts/` — проверки и генераторы (Node, без зависимостей)

| Скрипт | Назначение |
|---|---|
| `verify.js` | Один прогон всех проверок: банки + paid-sync + course-ref + `node --check` всех JS |
| `check-banks.js` | Инварианты банков (дубли стемов, перекос ключей A–D, length cue и др.) |
| `check-paid-sync.js` | Синхронность списков платных банков |
| `check-course-ref.js` | Целостность справочных панелей |
| `generate-bank-csv.js` | CSV платных банков для импорта в Supabase Table Editor |
| `generate-bank-seeds.js` | SQL-сиды банков (gitignored) |

## `.claude/agents/` и `.github/workflows/`

- Агентные лейны: `design.md`, `content.md`, `resources.md`, `marketing.md`,
  `smm.md` — каждый редактирует только свои файлы (правила — в `CLAUDE.md`
  основного репо; сводка будет в `21_AGENT_PROCESS.md`).
- Workflows: `claude-support.yml`, `claude-ticket-resolved.yml`,
  `facebook-post.yml`, `telegram-post.yml`.

## Ветки

- `main` — прод (GitHub Pages отдаёт как есть).
- `content-banks-src` — исходники всех банков, включая платные и
  незапущенный `la-business-law`.
- Прочие ветки на origin: UNKNOWN (в shallow-клоне видны только эти две и
  рабочая ветка сессии).

## Source References

- Полный листинг файлов: `find . -type f` по рабочей копии
  `lalianamen/llicena@main` (357 файлов, 2026-08-05)
- Роли файлов: header-комментарии самих файлов (`js/*.js`, `sw.js`,
  `scripts/*.js`, `supabase/functions/*/index.ts`, SQL-файлы) и `CLAUDE.md`
  основного репо (лейны, инварианты банков, ветка `content-banks-src`)
- Объёмы банков: пересчёт скриптом (Node `vm`) по `js/questions/*` на ветках
  `main` и `content-banks-src` (через `git archive origin/content-banks-src`)
- Счётчики: `wc -l css/*.css`, `grep -c "<loc>" sitemap.xml`
- Списки директорий: `ls` по `js/`, `js/samples/`, `docs/`, `es/`, `ru/`,
  `supabase/`, `.claude/agents/`, `.github/workflows/`
- `.gitignore`, `robots.txt`, `docs/content-audit/queue.json` (поле `_about`),
  `js/resources.js`, `js/reviews.js`, `js/sample-quiz.js` — прочитаны шапки
- Ветки: `git branch -r`, `git ls-tree origin/content-banks-src js/questions/`

## Verification Status

**Partially Verified.**

- Проверено: состав дерева (полный листинг), объёмы банков (пересчёт),
  счётчики строк/URL, назначения файлов, чьи шапки прочитаны.
- Взято из header-комментариев без построчной сверки: назначения модулей и
  Edge Functions; назначение SQL-файлов — по их именам и шапкам.
- Обновление 2026-08-05 (итерация «архитектура»): все SQL-файлы и Stripe-
  функции прочитаны полностью — их назначения подтверждены содержимым
  (`04_DATABASE.md`, `05_API.md`).
- `UNKNOWN`: прочие ветки origin (shallow-клон).
