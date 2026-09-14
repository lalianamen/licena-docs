# TASK — Качество данных: профили, сводка, платежи, источники, воронка, вход (список из 10 пунктов)

Последняя сверка: 2026-09-14

## Status

**CODE RELEASED — ждёт шагов владельца в Supabase** (2026-09-14, команда владельца
«заливаем»): `main` осн. репо @ `b5c7605` (fast-forward от `6320fe2`, чужих коммитов не было;
`verify.js` 144 OK на смерженном дереве). GitHub Pages отдаёт новые версии клиента
(`app.js?v=29`, `app-cabinet.js?v=73`, `stats.js?v=6`, …). Серверные шаги 2–10 Runbook
(SQL, деплой функций, настройки) на момент записи не выполнены — UNKNOWN.

Ранее: CODE READY_FOR_OWNER_STEPS — ветка @ `b5c7605` (от `main` @ `6320fe2`).

Запрос владельца (2026-09-13): список из 10 пунктов «что нужно изменить в Licena, по
приоритету» (составлен вне репозитория по выгрузкам владельца; исходные цифры «57 аккаунтов /
43 без профиля», «4 подписки в отчёте / 5 в таблице курсов» в репозитории отсутствуют), затем
«проверь это», затем «сделай список задач, исправь всё, что можно исправить без моего участия».

## Проверка пунктов (по коду `main` @ `6320fe2`, четыре параллельных прогона по областям)

| № | Пункт владельца | Вердикт | Факт (файл:строка на момент проверки) |
|---|---|---|---|
| 1 | Профили не создаются | **Подтверждено, дефект кода** | Единственный писатель `profiles` — кабинет (`js/app-cabinet.js:1691`); условие `!profRes.error && !profRes.data` недостижимо: `.single()` отвечает на отсутствующую строку ошибкой PGRST116 (supabase-js 2.110: только `.maybeSingle()` превращает 0 строк в `data: null` без ошибки). Триггера на `auth.users` нет. Код в таком виде с `e8211d3` (2026-08-13). Следствия: нет авто-выдачи `epa-608`/`contractor-business` (та же ветка); `stripe-checkout` терял `stripe_customer_id` (`update` в несуществующую строку); `stripe-portal` отвечал `no_customer`; `is_tester` (UPDATE) не применялся → тестеры в отчётах. |
| 2 | Сводка остановилась 10.09 | **Причина по репозиторию не определяется** | Дневные агрегаты пишет `marketing-aggregates` (cron `30 15 * * *`, сервисный ключ в теле `cron.job`), не `daily-stats`. В `main` есть `supabase/sql/diagnose-marketing.sql` (другая сессия, 2026-09-13) — запросы «не зарегистрирована / HTTP-ошибка / отработала, но не записала». Пересчёт: POST `{"days": N}` до 40, идемпотентно. Риск: подтверждённо задеплоена версия `daily-stats` 11.09 (719 строк) без проверки сервисной роли; если cron `daily-stats` создан через Dashboard UI, после деплоя новой версии получит 403. |
| 3 | 4 подписки vs 5 | **Расхождение заложено конструкцией** | Недельный отчёт: оплаченные счета Stripe за неделю (`app_events` `purchase`, `amount_paid > 0`); секция «Пользователи»: активные `user_courses` с `stripe_subscription_id` за всё время. Триал даёт $0-счёт и строку без `purchase`. `livemode` нигде не писался; колонки происхождения строки нет. Последние цифры в базе знаний: 19 аккаунтов / 6 платящих (`15_METRICS.md`, 2026-09-13). |
| 4 | GSC до 22.08 | **Не сломалось — не было расписания** | Единственный запуск `gsc-sync` 2026-08-25 вручную (окно до 08-22, лаг 3 дня). `cron-gsc-sync.sql` добавлен `0a4ccc1` (2026-09-13, другая сессия); применён ли в живой БД — UNKNOWN. |
| 5 | Источники, `other`, UTM | **Частично сделано, first-touch отсутствовал** | Классификатор (две копии, паритет `scripts/check-channel-parity.js`): `?src=`/`utm_source`, рефереры FB/IG/TG/TikTok/YouTube/поиск/ИИ. Канал `youtube` и `docs/marketing/utm-links.md` — `0a4ccc1`; порядок: `marketing-channels-youtube.sql` ДО деплоя `marketing-aggregates`. Сохранения источника до регистрации не было: ни ключа в localStorage, ни поля в `app_events`/`profiles`/метаданных регистрации. Обновлены ли био соцсетей — UNKNOWN. |
| 6 | Сумма дневных уникальных | **В недельной воронке — нет; в примере запроса и Looker — да** | `marketing_weekly_funnel`: `count(distinct device)` по окну недели. Сумма дневных уникальных — пример в комментарии `marketing-daily-aggregates.sql:147-152` и таблица каналов в `15_METRICS.md:99-112`; `reporting.channel_daily` в Looker без предупреждения. Стадии «подтверждение email» не было нигде. `who = coalesce(user_id, device)` считал одного человека дважды (до и после входа). |
| 7 | События без дублей и с одним пользователем | **Частично** | Все события есть (`sample_started` = начало практики, `first_answer`, `account_created`, `roadmap_*`, `checkout_started`). Ограничений уникальности в БД нет; `first_answer` защищён на устройстве; `checkout_started`/`pricing_viewed` — при каждом клике (в воронке считаются distinct за неделю). Связки устройство → пользователь не было; practice-страницы всегда слали `user_id: null`; события Stripe без устройства. |
| 8 | Просроченная ссылка входа | **Уже сделано** `0a4ccc1` | Панель `link-expired` с повторной отправкой (EN/ES/RU) в `main`. Вход парольный; речь о письмах подтверждения/сброса. Оставалось: язык в ссылке письма (сброс терял `?lang`), возврата на нужную страницу не было. |
| 9 | Свежесть данных | **Частично** | Дата снимка только у GSC; Bing/соцсети без даты; счётчики воронки подставляют 0; времени формирования нет; Looker-представления дней без `computed_at`; агрегатор пишет нулевые строки за дни без трафика. |
| 10 | Clarity | **Не проверяемо из репозитория** | Доступа к сессиям Clarity у Claude нет. |

## Сделано (Claude, ветка @ `b5c7605`; проверки: `verify.js` 144 OK, `test-marketing-core`, паритет каналов, `--check` всех `.ts`, SQL на локальном Postgres 16 с мок-схемой `auth.users`/таблиц, Playwright 22/22)

| Область | Изменение | Файлы |
|---|---|---|
| П1 Профили | `.maybeSingle()` в чтении профиля (ветка создания снова достижима); триггер `on_auth_user_created` → `handle_new_user()` (профиль из `raw_user_meta_data.name/lang`, курсы `epa-608` + `contractor-business`; ошибка не блокирует регистрацию); бэкфилл существующих `auth.users` | `js/app-cabinet.js`, **новый** `supabase/sql/profiles-trigger-backfill.sql` |
| П1 Stripe | `stripe-checkout`: `stripe_customer_id` через `upsert`; `stripe-portal`: при отсутствии id — поиск customer в Stripe по `metadata.user_id`, затем по email, сохранение в `profiles` | `supabase/functions/stripe-checkout/index.ts`, `stripe-portal/index.ts` |
| П3 Платежи | `stripe-webhook`: `meta.livemode` во всех событиях; воронка исключает `livemode = false` (строки без ключа считаются live); `reporting.subscriptions` — каждая строка `user_courses` с `kind` paid / trial / free / granted, `paid_invoices`, `is_tester` | `supabase/functions/stripe-webhook/index.ts`, `supabase/sql/marketing-weekly-funnel.sql`, `supabase/sql/reporting-looker.sql` |
| П5 Источник | `lp:first` `{src, ref, at}` при первом визите (обновляется один раз, если первый визит был без метки и внешнего реферера); `meta.src/ref` в `lpTrack` и событиях practice-страниц; `src/ref` в метаданных `signUp`; `marketing_weekly_sources`: для аккаунтов и покупок без просмотра — fallback на метаданные регистрации | `js/stats.js`, `js/pageview.js`, `js/sample-quiz.js`, `js/app.js`, `supabase/sql/marketing-weekly-funnel.sql` |
| П6/П7 Воронка | `who = coalesce(user_id, первый аккаунт устройства по page_views, device)`; колонка `email_confirmed` (по `auth.users.email_confirmed_at`, без тестеров) + строка «Подтвердили email» и конверсия «Аккаунт → подтверждение email» в письме; `user_id` на practice-событиях из сохранённого токена сессии | `supabase/sql/marketing-weekly-funnel.sql`, `supabase/functions/daily-stats/index.ts`, `js/sample-quiz.js` |
| П8 Вход | `?next=` (только относительный `.html` того же сайта, не лендинг) — `app.html`/`course.html` без сессии отправляют на лендинг с `next`, лендинг открывает форму входа и возвращает; `emailRedirectTo`/`redirectTo` писем с `?lang=` | `js/app.js`, `js/app-cabinet.js`, `js/app-course.js` |
| П9 Свежесть | Письмо: «Сформировано … UTC», даты снимка Bing / Instagram / Facebook, дата подписчиков, если старее конца недели; «нет данных» для колонки, которой SQL ещё не возвращает. Looker: `computed_at` в `daily_metrics`/`channel_daily`/`page_daily`/`state_snapshots` (+ предупреждение о сумме дневных уникальных), `reporting.freshness` (источник, последний день, время записи; GSC — динамически) | `supabase/functions/daily-stats/index.ts`, `supabase/sql/reporting-looker.sql` |
| Версии | `app.js?v=29`, `app-cabinet.js?v=73`, `app-course.js?v=77`, `stats.js?v=6` (8 страниц), `pageview.js?v=3` + `sample-quiz.js?v=10` (93 practice-страницы) | `*.html`, `practice/**` |

Не делалось (нельзя без владельца или вне запроса): причина остановки агрегатов (нужны
результаты `diagnose-marketing.sql` из живой БД); восстановление `stripe_customer_id` для уже
существующих аккаунтов происходит лениво при первом «Manage subscription» (stripe-portal);
`account_created` при повторной регистрации существующего email оставлен как есть (в воронке
аккаунты считаются по `auth.users`); Clarity.

## Runbook — шаги владельца (по порядку)

1. Команда «заливаем» → Claude мержит ветку в `main` (клиентские правки уходят на GitHub Pages).
2. Supabase → SQL Editor, файлы из репозитория целиком, в этом порядке:
   1. `supabase/sql/profiles-trigger-backfill.sql` — проверка в шапке файла (0 аккаунтов без профиля).
   2. `supabase/sql/tester-account.sql`, шаг 2 — `update … set is_tester = true` для тестовых
      email (раньше UPDATE был холостым); список тестовых email — в
      `supabase/sql/extend-tester-trials-oct31.sql`.
   3. `supabase/sql/marketing-channels-youtube.sql` — **до** деплоя `marketing-aggregates`.
   4. `supabase/sql/marketing-weekly-funnel.sql` (новая колонка `email_confirmed`, livemode, связка устройств).
   5. `supabase/sql/reporting-looker.sql` с **новым** `<REPORTER_PASSWORD>` (ротация после
      инцидента 2026-09-13 + новые представления); пароль в файл репозитория не вписывать.
   6. `supabase/sql/social-tokens.sql`; `supabase/sql/cron-tiktok-sync.sql` и
      `supabase/sql/cron-gsc-sync.sql` с подставленными `<PROJECT_REF>` = `vewhmndummfhnbxnrqya`
      и `<SERVICE_ROLE_KEY>`.
   7. `supabase/sql/diagnose-marketing.sql` — прислать вывод (задачи `cron.job`, последние
      запуски, свежесть `marketing_daily_metrics`) — по нему определяется причина п. 2.
3. Терминал (`git pull origin main`): `supabase functions deploy stripe-checkout`,
   `stripe-portal`, `stripe-webhook`, `daily-stats`, `marketing-aggregates` (после 2.3),
   `tiktok-auth --no-verify-jwt`, `tiktok-sync`.
4. Пересчёт агрегатов (сервисный ключ): `POST …/functions/v1/marketing-aggregates` с телом
   `{"days": 40}`; затем один `POST …/functions/v1/gsc-sync` с `{}`.
5. Если в выводе 2.7 задача `daily-stats` создана не из `cron-daily-stats.sql` (Dashboard UI) —
   пересоздать её этим файлом с сервисным ключом, иначе новая версия функции ответит 403.
6. Supabase → Authentication → URL Configuration → Redirect URLs: добавить
   `https://licena.us/index.html?lang=ru` и `https://licena.us/index.html?lang=es` (иначе ссылки
   писем откроют Site URL без языка — работать будут, язык возьмётся из браузера).
7. Looker Studio → источник → новый пароль (п. 2.5).
8. Stripe Dashboard → Developers → Webhooks: проверить, подписан ли endpoint и в тестовом
   режиме (UNKNOWN); с этого деплоя тестовые события помечаются `livemode = false` и в выручку
   не попадают.
9. Био Instagram / Facebook / TikTok / YouTube — ссылки по `docs/marketing/utm-links.md`.
10. TikTok: открыть `https://vewhmndummfhnbxnrqya.supabase.co/functions/v1/tiktok-auth` в
    браузере с входом как `licena_us`; затем `curl` `tiktok-sync` с сервисным ключом.

## Source References

- `lalianamen/llicena` ветка `claude/question-bank-generation-analysis-y47sk7` @ `b5c7605`
  (diff к `6320fe2`: 114 файлов, +550/−251); файлы перечислены в таблице «Сделано».
- Проверка пунктов: `js/app-cabinet.js`, `js/vendor/supabase-js-2.110.0.js` (обработка
  `isMaybeSingle`), `supabase/sql/*.sql`, `supabase/functions/*/index.ts`, `git log` (`e8211d3`,
  `0a4ccc1`, `e1a7790`, `5f869e7`), `15_METRICS.md`, `26_CHANGELOG.md`.
- Рендер-проверка: скрипт `fix-suite.mjs` (scratchpad сессии, не в репозитории): `?next` →
  форма входа (RU, 360px); `nextUrl` отвергает абсолютные, `//`, `javascript:`, не-`.html`,
  `index.html`; редиректы `app.html`/`course.html` без сессии; `lp:first` с `?src=` и
  устойчивость к последующему визиту; `meta.src` в `registration_started`; `signup` с
  `redirect_to=…?lang=ru` и метаданными `name/lang/src`; панель `link-expired` (ES) и
  `recover` с `redirect_to=…?lang=es`; practice-страница: `lp:first`, `meta.src`, `user_id`
  null/из токена; ноль ошибок консоли. Все вызовы Supabase перехвачены (в прод ничего не писалось).

## Verification Status

**Partially Verified** — код и SQL проверены локально (описано выше); применение в живой
Supabase, деплой функций, причина остановки агрегатов, тестовый режим Stripe-вебхука и
обновление био — UNKNOWN до шагов владельца.
