# 02 — Архитектура LICENA

Последняя сверка: 2026-08-05
Источник: код репозитория `lalianamen/llicena@main` (пути — относительно его корня).

## Общая схема

```
Браузер ──► GitHub Pages (main, статические файлы, домен licena.us)
   │            index.html / app.html / course.html / practice/* / guides/*
   │            vanilla HTML+CSS+JS, без сборки, без фреймворков
   │
   ├──► Supabase (проект vewhmndummfhnbxnrqya.supabase.co)
   │      • Auth (email+пароль, подтверждение почты, сброс пароля; implicit flow)
   │      • Postgres + RLS (profiles, user_courses, bank_questions, …)
   │      • Edge Functions (assistant, stripe-*, ticket-*, daily-stats)
   │      • Storage (загрузки в саппорт-тикеты)
   │
   └──► Stripe (через Edge Functions stripe-checkout / stripe-portal;
              доступ к курсу выдаёт только stripe-webhook)
```

- Хостинг: GitHub Pages отдаёт ветку `main` как есть (`CNAME` = licena.us,
  `.nojekyll`). Деплой = push в `main`. Vercel/Netlify-конфигурации в репо нет.
- Ключи фронтенда: URL проекта и publishable-ключ Supabase заданы в
  `js/supabase-client.js` (ключ публичный по дизайну Supabase; секретные ключи
  в репо не хранятся — `.gitignore` исключает `.env*`).
- Auth настроен на implicit flow — чтобы ссылки подтверждения почты и сброса
  пароля работали при открытии в другом браузере, чем тот, где регистрировались
  (комментарий в `js/supabase-client.js`).

## Страницы

| Страница | Роль |
|---|---|
| `index.html` | Лендинг + вход/регистрация (Supabase Auth); `?lang=es\|ru` переключает язык для всех, включая краулеры (`js/app.js`) |
| `app.html` | Кабинет: My courses / Catalog / Account; noindex, требует сессию |
| `course.html` | Плеер курса: блоки/секции, объяснения, режим экзамена; noindex |
| `about/`, `privacy.html`, `terms.html` | Публичные информационные страницы |
| `practice/<exam>/` — 20 стр. | SEO-страницы «per-exam» с публичным sample-квизом (`js/sample-quiz.js` + `js/samples/*` — отдельный публичный набор, не платный банк) |
| `guides/<topic>/` — 19 стр. | SEO-статьи-гайды по лицензиям (CA, AZ, EPA, NICET, OSHA, reciprocity) |
| `es/…`, `ru/…` | Полные зеркала `about`, `practice/*`, `guides/*` на испанском и русском (отдельные статические HTML) |

## Модули JS (по header-комментариям файлов)

| Файл | Роль |
|---|---|
| `js/supabase-client.js` | Инициализация клиента Supabase (implicit flow) |
| `js/app.js` (320 стр.) | Логика auth-страницы |
| `js/app-cabinet.js` (1518) | Кабинет: курсы, каталог (фильтр `LAUNCH_CATEGORIES=["construction"]`), аккаунт, Stripe checkout/portal, обработка возврата из Stripe |
| `js/app-course.js` (1766) | Плеер: банки, перемешивание вопросов, блоки (`buildBlockCards`/`activeBlock`), режим экзамена |
| `js/catalog.js` + `js/catalog/ca.js`, `js/catalog/az.js` | Реестр штатов и курсов; штат регистрирует категории через `registerState()` |
| `js/i18n.js` (302) / `js/i18n-app.js` (581) | Переводы лендинга / кабинета+плеера (EN/ES/RU) |
| `js/paths.js` | Данные «path wizard» — лицензия → набор экзаменов |
| `js/seo.js` (594) | Инжектор `<head>`: title/description/canonical/robots/OG/hreflang/JSON-LD per page + per lang |
| `js/pwa.js` + `sw.js` | PWA: регистрация service worker; network-first, кэш `licena-v1`, precache `/` и `/app.html`; кросс-доменные запросы (Supabase) не перехватываются |
| `js/devices.js` | Клиент анти-шеринга: тонкий вызов RPC `register_device`, fail-open |
| `js/onboarding.js` | Первичные туры-подсказки (coach marks), по одному на зону, показ один раз |
| `js/stats.js` / `js/pageview.js` | First-party пейдж-вью в `public.page_views` (без cookies и третьих сторон); `pageview.js` — облегчённый вариант для `/practice/*` без загрузки supabase-js |
| `js/honest-chances.js` (1136) | Блок «Honest chances» в шапке курса |
| `js/support.js` | Виджет AI-саппорта; все вызовы через Edge Function `assistant` |
| `js/course-blocks.js` | Метаданные блоков и локализация названий секций |
| `js/course-ref.js` (1575) | Справочная панель курса: официальные источники, изменения, расчёты |
| `js/bank-updates.js` + `js/bank-updates-render.js` | Журнал обновлений банков (макс. 6 записей) + его рендер в кабинете |
| `js/resources.js` + `js/resources-render.js` | Данные официальных ссылок + рендер слота «Official resources» |
| `js/reviews.js` | Одобренные отзывы на лендинге; при отсутствии — секция скрывается (никаких плейсхолдеров) |
| `js/sample-quiz.js` + `js/samples/*` (20 файлов) | Публичные sample-квизы для `/practice/*` |
| `js/guides/*` (3 файла) | Контент курсов-гайдов (`COURSE_GUIDE[id]`, EN/ES/RU) |
| `js/vendor/supabase-js-2.110.0.js` | Единственная vendor-библиотека |

CSS: `css/styles.css` (582 строк — база+токены+auth), `cabinet.css` (664),
`course.css` (514), `exam.css` (282). CSP запрещает inline-скрипты
(комментарий в `js/pwa.js`).

## Данные и доступ к банкам

- Формат вопроса: `{id, sec, block?, q, opts[4], correct, re}`;
  банк — `window.COURSE_REGISTRY[courseId]` (`js/questions/*.js`,
  схема — `data/questions.example.json`).
- **Бесплатные банки** — статические файлы в `main` (лид-магнит, осознанное
  решение). **Платные банки** живут только в Supabase `public.bank_questions`
  под RLS; плеер читает таблицу и восстанавливает те же глобалы
  (`js/app-course.js:48–107,142–170`). Исходники платных банков — ветка
  `content-banks-src`; в `main` их нет с 2026-07-29.
- Переводы: RU/ES-файлы — оверлеи, соединяются с EN по индексу опций;
  `correct` есть только в EN.
- Таблицы Postgres, которые трогает клиент: `profiles`, `devices`,
  `user_courses`, `course_trials`, `bank_questions`, `page_views`, `reviews`
  (грep по `from("…")` в `js/*.js`); RPC: `register_device`, `start_trial`.
  Прочие таблицы из SQL-файлов: `support_tickets`, `social_stats`
  (подробности схемы — в `04_DATABASE.md`, ещё не создан).

## Режим экзамена (`js/app-course.js`)

- По умолчанию: 120 вопросов, проходной 75 %, 210 минут, экзамен доступен от
  120 вопросов в банке (`EXAM_COUNT`, `EXAM_PASS_PCT`, `EXAM_MINUTES.DEFAULT`,
  `EXAM_MIN_BANK`, строки 856–859).
- `EXAM_FORMATS`: `epa-608` — 25 вопросов с каждого блока, порог 72 % в каждой
  секции отдельно (как на реальном прокторинге EPA 608); `osha-construction` —
  20 вопросов, порог 70 % на весь тест.

## Платежи

- `stripe-checkout` (Edge Function): подписочная Checkout-сессия $20/мес за
  курс (`PRICE_CENTS=2000`, `CURRENCY=usd`), карта из 14 платных курсов;
  функция никогда не пишет `user_courses`.
- `stripe-webhook`: единственный, кто выдаёт доступ (`user_courses`) после
  оплаты. `stripe-portal`: отмена/карта/инвойсы.
- Триал: RPC `start_trial` + таблица `course_trials` (3 дня, один раз на
  (user, course)); истёкшие подписки гасит ежедневный cron
  `expire-subscriptions` (`supabase/sql/trial-3day.sql`).

## Анти-шеринг (`docs/access-control.md`, `supabase/devices_anti_sharing.sql`)

Серверная функция `register_device()` (security definer): **5 устройств на
аккаунт** по per-device токену (не по IP); при достижении лимита замена
устройства — не чаще 1 раза в 30 дней, вытесняется самое неактивное. Клиент
fail-open: сбой бэкенда никого не блокирует.

## Саппорт и автоматизация

- AI-ассистент: Edge Function `assistant`, модель `claude-sonnet-4-6`,
  веб-поиск на стороне функции; ключ API в браузер не попадает
  (`js/support.js`, `supabase/functions/assistant/index.ts`).
- Тикеты: `support_tickets` + Storage для вложений + триггеры
  `ticket_issue`/`ticket_email` (создание GitHub issue, письма) и Edge
  Functions `ticket-issue`, `ticket-email`, `ticket-status`.
- GitHub Actions (`.github/workflows/`): `claude-support.yml` (issue opened/
  labeled → Claude работает над тикетом), `claude-ticket-resolved.yml`
  (PR closed → уведомление пользователю), `facebook-post.yml` и
  `telegram-post.yml` (push файлов в `docs/smm/queue-fb/*.txt` /
  `docs/smm/queue/*.txt` в `main` → автопост в соцсеть).
- `daily-stats` (Edge Function) + `supabase/sql/cron-daily-stats.sql` —
  ежедневная статистика.

## SEO-инфраструктура

- `js/seo.js` — единственный владелец crawl-facing `<head>` (SEO-копия хранится
  в нём, не в общих i18n).
- `robots.txt`: всё публичное открыто; `Allow: /js/samples/`; `Disallow:`
  `/supabase/`, `/docs/`, `/js/questions/`, `/js/guides/` (crawl-уровневая
  защита; реальная защита платного контента — БД под RLS).
- `sitemap.xml`: 126 URL. Google Search Console подтверждён
  (`googlec4665304f2eceeb0.html`). Стороннего JS-аналитикса нет (grep по
  gtag/googletagmanager — пусто); аналитика first-party (`page_views`).

## Верификация (нет тест-фреймворка — vanilla static)

`node scripts/verify.js` = `check-banks.js` (инварианты банков) +
`check-paid-sync.js` + `check-course-ref.js` + `node --check` всех
не-vendor JS. Остальная проверка — рендер страниц (Playwright-скриншоты,
desktop + 360px, EN + RU) по чек-листу из `CLAUDE.md` основного репо.
