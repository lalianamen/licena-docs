# 05 — API и интерфейсы

Последняя сверка: 2026-08-05
Источник: код `lalianamen/llicena@main`. Собственного REST-бэкенда у проекта
нет — API-поверхность состоит из Supabase (PostgREST + Auth + RPC + Edge
Functions + Storage) и внешних сервисов, вызываемых с сервера/из CI.

## 1. Клиент → Supabase PostgREST (через `js/vendor/supabase-js-2.110.0.js`)

Клиент инициализируется в `js/supabase-client.js` (URL проекта
`https://vewhmndummfhnbxnrqya.supabase.co`, publishable-ключ, auth
`flowType:"implicit"`). Операции клиента по таблицам (файл:строка вызова):

| Таблица | Операции клиента | Где |
|---|---|---|
| `profiles` | select/insert/update своих строк (имя, язык, `is_tester`, `stripe_customer_id` читается) | `js/app-cabinet.js:1332,1402,1456,1467,1480,1674` |
| `user_courses` | select своих, upsert/update (free tier), delete | `js/app-cabinet.js:1112,1140,1177,1457,1473,1481,1692`; `js/app-course.js:97,734` (проверка доступа из плеера) |
| `course_trials` | select своих (решение «Trial vs Subscribe») | `js/app-cabinet.js:1460` |
| `bank_questions` | select с `.range()`-пагинацией (платные банки, EN + перевод) | `js/app-course.js:72,170` |
| `reviews` | insert (со status `pending`), select approved на лендинге | `js/app-cabinet.js:1240`; `js/reviews.js:67` |
| `page_views` | insert (бикон) | `js/stats.js:45` |
| `devices` | select своих (список устройств в аккаунте) | `js/app-cabinet.js:920` |

Особый случай: `js/pageview.js` шлёт INSERT в `page_views` **прямым REST-запросом**
(fetch к PostgREST) без загрузки supabase-js — для лёгких страниц `/practice/*`.

## 2. Клиент → RPC (Postgres-функции)

| RPC | Вызов | Контракт |
|---|---|---|
| `register_device(p_token, p_confirm=false)` | `js/devices.js:22,31` | Возвраты: `'ok' \| 'add' \| 'swap' \| 'blocked' \| 'unauthenticated' \| 'error'`. `p_confirm=false` — dry-run для UI; `true` — фактическая регистрация/замена. Определение: `supabase/devices_anti_sharing.sql` |
| `start_trial(p_course)` | `js/app-cabinet.js:1173` | Возвраты: `'ok' \| 'trial_used' \| 'already_active' \| 'not_paid_course' \| 'unauthenticated'`. Определение: `supabase/sql/trial-3day.sql` |

Server-side-only функции (revoke от клиентских ролей): `daily_signups(days)`,
`recent_signups(days)` (`supabase/sql/report-kpi.sql`) — используются функцией
`daily-stats`.

## 3. Supabase Auth

- Регистрация/вход по email+паролю, обязательное подтверждение почты, сброс
  пароля — стандартные эндпоинты Supabase Auth через supabase-js (`js/app.js`).
- Flow: **implicit** — токен приходит в URL письма, чтобы ссылки работали в
  другом браузере, чем при регистрации (`js/supabase-client.js`, комментарий).
- `stripe-checkout`/`stripe-portal` проверяют вызывающего сами: сначала подпись
  JWT по JWKS проекта (`/auth/v1/.well-known/jwks.json`, npm `jose@5`), затем
  fallback `GET /auth/v1/user` — работает даже когда сессия убита политикой
  single-session (`supabase/functions/stripe-checkout/index.ts:60–91`).

## 4. Edge Functions (все — POST `https://<proj>.supabase.co/functions/v1/<name>`)

| Функция | Auth | Вход | Выход / действие |
|---|---|---|---|
| `assistant` | Verify JWT (invoke с anon key + сессией) | conversation (≤40 сообщений, ≤8000 симв./сообщение), контекст штата, signed URLs скриншотов | Прокси к Claude API, модель `claude-sonnet-4-6`, tool-use цикл ≤6 ходов: ответы с web search + запись тикетов в `support_tickets` через tool. Ключ API только на сервере |
| `stripe-checkout` | JWT (JWKS/fallback, см. §3) | `{ course_id }` | Проверяет `course_id` в карте `PAID` (14 курсов) → 400 `unknown_or_free_course`; отсекает живую подписку → 409 `already_active`; лениво создаёт Stripe customer (пишет `profiles.stripe_customer_id`); создаёт subscription Checkout Session ($20/мес, metadata `{user_id, course_id}` на сессии И подписке) → `{ url }`; redirect обратно на `app.html?checkout=success\|cancel` |
| `stripe-portal` | JWT (как выше) | `{}` | Создаёт Billing-Portal-сессию Stripe → `{ url }` (отмена, карта, инвойсы) |
| `stripe-webhook` | **Verify JWT OFF**; подпись Stripe (`stripe-signature` + `STRIPE_WEBHOOK_SECRET`) | события Stripe | Единственный писатель платных прав: `checkout.session.completed` и `invoice.paid` → upsert `user_courses` (active, `expires_at` = конец периода + 2 дня grace, `auto_renew`, `stripe_subscription_id`); `customer.subscription.updated` → зеркалит `auto_renew`; `customer.subscription.deleted` → inactive. Ошибка обработчика → 500, чтобы Stripe ретраил (записи идемпотентны) |
| `ticket-email` | заголовок `x-ticket-secret` (общий секрет с триггером) | `{type, record, old_record}` от pg_net-триггера | Транзакционные письма через Resend от `LICENA <noreply@licena.us>`: received / done / rejected, локали EN/ES/RU |
| `ticket-issue` | `x-ticket-secret` | `{type:'INSERT', record}` (только kind request/complaint) | Создаёт GitHub issue в `lalianamen/llicena` (fine-grained PAT, label `support`; label `from-chat` ставится владельцем вручную — greenlight-политика) + уведомление владельцу через Resend |
| `ticket-status` | `x-ticket-secret` | `{ticket_id, status}` (default `done`) | Привилегированный UPDATE статуса тикета (service role мимо RLS); смена статуса сама триггерит письмо. Вызывается workflow'ом `claude-ticket-resolved` при merge PR |
| `daily-stats` | Verify JWT ON (cron шлёт service-role JWT) | `{}` | Читает `page_views` service role'ом; агрегаты по дням Pacific, разбивка по каналам (`?src=`-метка приоритетнее реферера: fb/ig/tg/tt/flyer/qr), 7-дневная таблица, топ страниц; 1-го числа — месячный rollup; письмо через Resend получателям из `STATS_EMAIL` |

Секреты функций (имена; значения — в Supabase Edge Functions → Secrets):
`CLAUDE_API_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`,
`RESEND_API_KEY`, `TICKET_WEBHOOK_SECRET`, `GH_ISSUE_TOKEN`, `OWNER_EMAIL`,
`STATS_EMAIL`; `SUPABASE_URL` / `SUPABASE_ANON_KEY` /
`SUPABASE_SERVICE_ROLE_KEY` инжектируются платформой.

## 5. Внутренние вебхуки БД (pg_net)

Триггеры на `support_tickets` шлют `net.http_post` в `ticket-email` (INSERT и
UPDATE→done/rejected) и `ticket-issue` (INSERT request/complaint) — заголовок
`Authorization: Bearer <publishable key>` только для прохождения Edge-гейтвея,
реальная авторизация — `x-ticket-secret`
(`supabase/ticket_email_trigger.sql`, `ticket_issue_trigger.sql`).

## 6. Внешние API

| Сервис | Кто вызывает | Что используется |
|---|---|---|
| Stripe API (npm `stripe@17`) | `stripe-checkout`, `stripe-portal`, `stripe-webhook` | `customers.create`, `checkout.sessions.create` (mode subscription, price_data $20/usd/month), `billingPortal`-сессии, `subscriptions.retrieve/update`, верификация вебхуков |
| Anthropic Claude API (npm `@anthropic-ai/sdk`) | `assistant` | model `claude-sonnet-4-6`, web search, tools (запись тикетов), лимиты: 6 ходов, 40 сообщений, 8000 символов |
| Resend | `ticket-email`, `ticket-issue`, `daily-stats` | транзакционные письма от `noreply@licena.us` |
| GitHub REST API | `ticket-issue` (создание issue); workflow `claude-support` (`anthropics/claude-code-action@v1`, секрет `ANTHROPIC_API_KEY`) | issues в `lalianamen/llicena` |
| Telegram Bot API | workflow `telegram-post.yml` | `POST https://api.telegram.org/bot<TOKEN>/sendMessage` при push `docs/smm/queue/*.txt` |
| Facebook Graph API v23.0 | workflow `facebook-post.yml` | `POST /me/feed` при push `docs/smm/queue-fb/*.txt` |

## 7. Данные, отдаваемые статикой (без API)

Бесплатные банки (`js/questions/*`), каталог, i18n, sample-квизы
(`js/samples/*`), гайды (`js/guides/*`), ресурсы (`js/resources.js`) — обычные
`<script>`-файлы с GitHub Pages; для них API нет.

## Source References

- `js/supabase-client.js`, `js/devices.js`, `js/stats.js`, `js/pageview.js`,
  `js/reviews.js`, `js/app.js`, `js/app-cabinet.js`, `js/app-course.js`
  (вызовы `from()`/`rpc()`/`functions.invoke()` — grep с номерами строк)
- `supabase/functions/stripe-checkout/index.ts`,
  `stripe-webhook/index.ts`, `stripe-portal/index.ts` — прочитаны полностью;
  `assistant/index.ts` (строки 1–40 + grep MODEL/env),
  `ticket-issue/index.ts` (строки 1–60 + grep env),
  `ticket-email/index.ts` (строки 1–30), `ticket-status/index.ts` (строки 1–30),
  `daily-stats/index.ts` (строки 1–45 + grep env)
- `supabase/devices_anti_sharing.sql`, `supabase/sql/trial-3day.sql`,
  `report-kpi.sql`, `cron-daily-stats.sql`,
  `supabase/ticket_email_trigger.sql`, `ticket_issue_trigger.sql`
- `.github/workflows/claude-support.yml`, `telegram-post.yml`,
  `facebook-post.yml` (строки вызовов API)

## Verification Status

**Partially Verified.**

- Проверено чтением кода: все контракты RPC, три Stripe-функции целиком,
  триггерные вебхуки, список секретов (по grep `Deno.env.get`), операции
  клиента по таблицам, лимиты assistant.
- Прочитано частично (шапки + целевые grep): `assistant` (полный tool-набор и
  формат ответа не документированы — см. будущий `06_FUNCTIONS.md`),
  `ticket-email` (полный словарь писем), `daily-stats` (полная логика KPI),
  `ticket-issue` (хвост файла), тела workflow-файлов.
- UNKNOWN: настроенные в Stripe Dashboard события вебхука (код перечисляет
  ожидаемые 4), фактические значения секретов, активирован ли Customer Portal.
