# TASK — Analytics funnel for the owner's metrics tracker (Licena_All_Metrics_Tracker.xlsx)

Последняя сверка: 2026-09-11

## Status

**CODE RELEASED — ждёт шагов владельца в Supabase** (2026-09-11, команда «да мерж, а я займус базой»):
`main` осн. репо @ `0f9f663` (merge `origin/main` @ `97d1c90` ← ветка `f80b96c`, конфликтов не было;
проверки на смерженном дереве: `verify.js` 139, паритет, `test-marketing-core` 48/48, TS-синтаксис,
Playwright 14/14, вехи 6/6). GitHub Pages отдаёт `ga.js?v=2`, `stats.js?v=5`, `app.js?v=27`,
`landing-extras.js?v=4`, `sample-quiz.js?v=9`; байты семи JS-файлов совпадают с репозиторием —
события с сайта уже пишутся в `app_events` и GA4. Шаги 2–6 Runbook (SQL, функции, секрет, cron)
выполняет владелец; на момент записи не подтверждены.

Предыдущий статус: READY_FOR_OWNER_STEPS — код в ветке `claude/question-bank-generation-analysis-y47sk7`
@ `f80b96c` (от `main` @ `b35238f`).

Запрос владельца (2026-09-11): «подготовь все что сможешь, а я пока спать утром встану доделаем» —
после разбора трекера (5 листов, справочник из 20 метрик) и ответа «что реализовано, что нет».

## Что сделано (без участия владельца)

### 1. События воронки — первопартийно (`public.app_events`) и в GA4

| Метрика трекера | Событие (имя одинаково в `app_events` и GA4) | Где срабатывает |
|---|---|---|
| landing_view | `page_view` GA4 (авто) + `page_views` first-party — нового события нет | все страницы |
| hero_cta_click | `hero_cta_click` {cta} | `js/landing-extras.js` — кнопки `.hero .cta-row a` |
| sample_started | `sample_started` {course, page} | демо на главной (`landing-extras.js`), practice-страницы (`sample-quiz.js`, первый ответ) |
| sample_answered | `sample_answered` {course, page, q, correct} | там же, каждый ответ |
| explanation_viewed | `explanation_viewed` {course, page, q} | объяснение ≥ 50 % в зоне видимости (IntersectionObserver), один раз на вопрос |
| registration_started | `registration_started` {page} | `js/app.js` — открытие панели регистрации, один раз на просмотр |
| account_created | `account_created` {lang} | `js/app.js` — успешный `signUp` (истина — `auth.users`; уже зарегистрированный e-mail тоже попадает, намеренно) |
| first_answer / 20 / 100 | `first_answer`, `questions_20_completed`, `questions_100_completed` {course, answered} | `js/app-course.js` — один раз на курс и устройство (`lp:fnl:<course>`), после рестарта не повторяется |
| pricing_viewed | `pricing_viewed` {page} / {course, trial} | секция `#pricing` на главной (≥ 30 % видна); модал оплаты в кабинете для платного курса |
| checkout_started | `checkout_started` {course} | `js/app-cabinet.js` перед вызовом `stripe-checkout` |
| — | `checkout_completed` / `checkout_cancelled` {course} | возврат из Stripe (`app.html?checkout=success|cancel`) |
| purchase | `purchase` {course, amount_paid, currency, billing_reason, stripe_event} | **сервер** `stripe-webhook`, `invoice.paid` с `amount_paid > 0` (пробный период без списания — не покупка) |
| — | `subscription_started` {course, trial, amount_total} | `checkout.session.completed` |
| subscription_cancelled | `subscription_cancelled` {course, at_period_end, reason, feedback} | переход auto-renew → off или немедленная отмена |
| — | `subscription_ended` | `customer.subscription.deleted` |

Механика: `js/ga.js` v2 — `?src=<метка>` уходит в GA4 как `campaign_source` (medium `src`), чтобы
трафик из соцсетей и флаеров не считался «direct»; хелпер `window.lpGa`. `js/stats.js` v5 —
`lpTrack` зеркалит каждое именованное событие в GA4 (примитивные поля meta → параметры).
Practice-страницы не грузят supabase-js — `sample-quiz.js` v9 шлёт события raw REST в
`app_events` (как раньше `sample_result`). Clarity-события не тронуты.

Версии: `ga.js?v=2` на 184 страницах, `sample-quiz.js?v=9` на 82, `stats.js?v=5`, `app.js?v=27`,
`landing-extras.js?v=4`, `app-cabinet.js?v=71`, `app-course.js?v=74`.

### 2. Метки по постам

Классификатор каналов (`daily-stats/index.ts` и `marketing-aggregates/core.ts`, блок
байт-идентичен — `scripts/check-channel-parity.js`) понимает метки вида
`<канал>-<что угодно>`: `ig-2026-09-12-c20` → instagram, `tt-video7` → tiktok,
`flyer-sba` → flyer, `fb-…`, `tg-…`. Неизвестный префикс → `other`.
Формат для роликов: `?src=<канал>-<дата>-<ролик>`; сырая метка хранится в `page_views.path`,
так что разрез «по ролику» считается SQL-запросом по `path like '%src=ig-2026-09-12-c20%'`.
Пересчёт истории не нужен: раньше префиксные метки не использовались.

### 3. Недельная воронка в письме

`supabase/sql/marketing-weekly-funnel.sql` — функция `marketing_weekly_funnel(p_week_start date)`
(security definer только ради `auth.users`; EXECUTE у service_role). Возвращает колонки листа
«Воронка по неделям» за PT-неделю Пн–Вс: посетители (device-токены), с 2+ просмотрами,
sample_started / answered / explanation_viewed, registration_started, аккаунты (без тестеров),
first_answer / 20 / 100, pricing_viewed, checkout_started / completed, покупки (уникальные
события Stripe), выручка в центах, отмены, D1 (устройства с ≥ 2 днями активности за неделю),
D7 (были и на прошлой неделе).
`daily-stats` по понедельникам добавляет секцию «Воронка за неделю»: таблица счётчиков,
конверсии по формулам листа (доля пробы, проба → аккаунт, аккаунт → первый ответ / 20 /
покупка, 20 → оплата, checkout → покупка, доход на покупателя), основной источник (канал
первого визита), Bing показы/клики из снимка. Тело `{"week":"YYYY-MM-DD"}` (PT-понедельник)
включает секцию в любой день — для проверки.

### 4. Bing

`supabase/functions/bing-sync/index.ts` — Bing Webmaster JSON API (`GetRankAndTrafficStats`,
`GetQueryStats`, `GetPageStats`, `GetCrawlStats`) → `public.bing_snapshots`
(`supabase/sql/bing-snapshots.sql`, service-only). Даты `/Date(ms)/` нормализуются в ISO.
Cron: `supabase/sql/cron-bing-sync.sql` (понедельник 14:30 UTC, до письма).
**AI Performance (цитирования Copilot) API не имеет** (Microsoft, 02/2026: «позже в 2026») —
лист «SEO и ИИ» в части ИИ остаётся ручным экспортом CSV из Bing.

## Проверки (на ветке)

`node scripts/verify.js` — 139 файлов; `check-channel-parity` OK; `test-marketing-core` 48/48
(+5 на префиксные метки); синтаксис TS (`node --experimental-strip-types --check`) для
daily-stats, bing-sync, stripe-webhook, core; Playwright с перехватом `dataLayer` и POST в
`app_events`: главная (campaign_source из `?src=`, hero_cta_click, демо-вопрос: started /
answered / explanation_viewed один раз, pricing_viewed, registration_started один раз, GA4 не
дублируется) и practice-страница (три события с course/page/q, второй вопрос без нового
started) — 14/14, без ошибок консоли; логика вех курса (1/20/100, персистентность, без
повтора после рестарта) — 6/6 в браузерном контексте (страница курса требует входа).

## Правка 2026-09-11: живая схема `bing_snapshots` (`main` @ `a98bb16`)

При проверке выяснилось, что `public.bing_snapshots` существовала в Supabase до этой задачи
(в карточке агрегатов от 2026-08-29 она уже перечислена) со схемой `id, fetched_at, site_url,
dataset, start_date, end_date, row_count, payload` и содержит строки `rank_traffic`,
`page_stats`, `crawl_stats`, `crawl_issues` (fetched_at 2026-09-11 08:20 UTC — от более ранней
синхронизации, код которой в репозитории отсутствует; UNKNOWN, что именно её пишет).
Первая версия `bing-sync` писала колонку `site` и датасеты `Get*` — вставки падали бы.
Исправлено: функция пишет `site_url`, `start_date`/`end_date`, датасеты `rank_traffic`,
`query_stats`, `page_stats`, `crawl_stats`, `crawl_issues`; секция письма читает `rank_traffic`
в обоих форматах строк; `bing-snapshots.sql` приведён к живой схеме (на живой базе — no-op).
**Требуется повторный деплой `bing-sync` и `daily-stats`** из `main` @ `a98bb16`.
Проверка пути записи `app_events` 2026-09-11: вставка через публичный ключ — HTTP 201
(тестовая строка `sample_started` с `meta.test = true`, удаляется владельцем).

## Подтверждено в production 2026-09-11

- `bing-sync` (`main` @ `c50682c`, принимает legacy `service_role` и `sb_secret_…` через
  пробу admin-эндпоинта): ручной запуск владельца 18:05 UTC — `ok:true`, датасеты
  `rank_traffic` 26 строк, `query_stats` 44, `page_stats` 26, `crawl_stats` 24, `crawl_issues` 0.
- Развёрнуты: `daily-stats` (новая версия отвечает полем `weekly`), `marketing-aggregates`,
  `stripe-webhook`; SQL `marketing_weekly_funnel` и таблица `bing_snapshots` на месте
  (anon получает permission denied). Не подтверждены на момент записи: повторный деплой
  `daily-stats` с `a98bb16` (чтение `rank_traffic`), cron `bing-sync`, тестовое письмо.

## Runbook — шаги владельца (утро)

1. **Выкладка кода.** Команда «заливаем» — Claude мержит ветку в `main` (или сделайте это
   сами: `git merge claude/question-bank-generation-analysis-y47sk7`). После деплоя
   GitHub Pages события с сайта начинают приходить в `app_events` и GA4 сразу.
2. **SQL Editor (Supabase), по порядку:**
   - `supabase/sql/marketing-weekly-funnel.sql`
   - `supabase/sql/bing-snapshots.sql`
3. **Edge Functions — передеплоить:** `daily-stats`, `marketing-aggregates`, `bing-sync`
   (Verify JWT ON), `stripe-webhook` (`--no-verify-jwt`, как раньше).
4. **Секрет:** `BING_API_KEY` — Bing Webmaster Tools → Settings → API access → Generate API Key
   (один ключ на пользователя). Положить в Supabase → Edge Functions → Secrets.
5. **Cron:** `supabase/sql/cron-bing-sync.sql` с подставленными `<PROJECT_REF>` и
   `<SERVICE_ROLE_KEY>` (в SQL Editor, не коммитить).
6. **Проверка:**
   - `bing-sync` вызвать вручную с service-role ключом → в `bing_snapshots` 4 строки;
   - `daily-stats` вызвать с телом `{"week":"<прошлый понедельник>"}` → в письме секция
     «Воронка за неделю» (пока с нулями по новым событиям — данные копятся с момента выкладки);
   - в GA4 → Realtime после выкладки видны `sample_answered`, `hero_cta_click`.
7. **GA4 (по желанию):** отметить как ключевые события `sample_started`, `account_created`,
   `checkout_started`, `checkout_completed`; в отчётах по источникам появится
   `campaign_source = <метка>`.
8. **Публикации:** метки роликов в формате `?src=<канал>-<дата>-<ролик>`.

## Что остаётся вручную

Просмотры/досмотры соцсетей (инсайты платформ), Telegram-подписчики уже в `social_stats`;
Bing AI Performance (нет API); заполнение листов из письма и панелей. Выручка в письме — по
оплаченным счетам Stripe (после первого списания), не по созданным сессиям.

## Source References

Осн. репо, ветка `claude/question-bank-generation-analysis-y47sk7` @ `f80b96c`: `js/ga.js`,
`js/stats.js`, `js/landing-extras.js`, `js/app.js`, `js/app-course.js`, `js/app-cabinet.js`,
`js/sample-quiz.js`, `supabase/functions/stripe-webhook/index.ts`,
`supabase/functions/daily-stats/index.ts`, `supabase/functions/marketing-aggregates/core.ts`,
`supabase/functions/bing-sync/index.ts`, `supabase/sql/marketing-weekly-funnel.sql`,
`supabase/sql/bing-snapshots.sql`, `supabase/sql/cron-bing-sync.sql`,
`scripts/test-marketing-core.mjs`; трекер владельца `Licena_All_Metrics_Tracker.xlsx`
(лист «Справочник метрик»); Microsoft Learn — Bing Webmaster API (getting-access,
GetRankAndTrafficStats), Microsoft Q&A об отсутствии API AI Performance (02/2026).

## Verification Status

**Partially Verified** — код и тесты проверены прогоном на ветке; поведение в production
(SQL, функции, cron, секрет) не проверено — ждёт шагов владельца.
