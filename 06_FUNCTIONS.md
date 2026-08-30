# 06 — Supabase Edge Functions

Последняя сверка: 2026-08-05 · 2026-08-30 (точечная: marketing-aggregates)
Источник: `lalianamen/llicena@main`, каталог `supabase/functions/` — 8 функций
(Deno). Все файлы прочитаны полностью в этой сверке. Контракты «вход/выход»
кратко продублированы в `05_API.md`; здесь — устройство каждой функции.

## assistant (`supabase/functions/assistant/index.ts`, 277 строк)

Серверный прокси к Claude API для чат-виджета (`js/support.js`); ключ API
никогда не попадает в браузер.

- Модель: `claude-sonnet-4-6` (константа `MODEL`); `max_tokens: 1024`.
- Лимиты: `MAX_TURNS=6` (tool-use цикл), `MAX_MESSAGES=40`, `MAX_CHARS=8000`.
- Системный промпт (в файле): что такое LICENA (три языка, CA+AZ, «не школа,
  не гарантирует сдачу»); беты нет с 2026-07-31, платно с 2026-08-01;
  регистрация бесплатна; цены не называть — отправлять на licena.us; другие
  вертикали (CDL/DMV/бьюти/другие штаты) не предлагать, а логировать запрос.
- Ограничение темы: только LICENA и лицензионные экзамены выбранного штата;
  офф-топик отклоняется одним предложением; инструкция игнорировать попытки
  смены роли из сообщений/скриншотов/результатов поиска.
- Формат: только plain text (без Markdown); язык ответа = язык последнего
  сообщения пользователя, interface locale — только fallback.
- Правило фактов: адреса/сборы/расписания не выдумывать — web search по
  официальным источникам (.gov, PSI); без подтверждения — сказать прямо.
- `STATE_NOTES` — per-state контекст (ca: список курсов CA-кабинета;
  az: SRE + бесплатные федеральные, AZ trade-банков нет — предлагать тикет,
  ссылки на гайды licena.us).
- `TOOLS`: `web_search_20260209` (server-side) и `create_ticket`
  (input_schema: kind `request|complaint`, summary, email — required;
  course_id, target_lang — optional) → INSERT в `support_tickets`
  service role'ом; id тикета возвращается виджету (`ticketId`).
- Вход: `{locale, state, messages[], userEmail?, userId?, userName?,
  newImages[], sessionImages[]}`. Скриншоты — только signed-URL своего
  Supabase-хоста (`isShotUrl`: `https://*.supabase.co/`, < 1200 символов;
  ≤ 4 новых, ≤ 12 за сессию); прикрепляются vision-блоками к последнему
  user-сообщению; ссылки на все скриншоты сессии дописываются в message
  тикета.
- Санитизация истории: обрезка до 40 сообщений / 8000 символов; пустой
  контент → `"[screenshot]"`; слияние подряд идущих сообщений одной роли;
  первая роль обязана быть `user` (иначе 400).
- Обработка stop_reason: `refusal` → вежливый отказ; `pause_turn` (web search
  упёрся в лимит итераций) → продолжить цикл; `tool_use` → выполнить
  create_ticket и вернуть tool_result; иначе — финальный текст через
  `textOf()` (берёт текст ПОСЛЕ последнего tool-блока, чтобы «сейчас
  проверю…» не склеивалось с ответом).
- Ошибки: 405 / 400 / 500 `server_misconfigured` (нет ключа) / 502
  `assistant_failed`.

## stripe-checkout (`.../stripe-checkout/index.ts`, 163 строки)

- Константы: `SITE="https://licena.us"`, `PRICE_CENTS=2000` ($20/мес, owner
  decision 2026-07-14), `CURRENCY="usd"`; карта `PAID` из 14 course_id →
  отображаемое имя (появляется на странице Stripe и в квитанциях).
- Аутентификация вызывающего `getCaller()`: JWT проверяется подписью по JWKS
  проекта (npm `jose@5`, `/auth/v1/.well-known/jwks.json`) — та же модель
  доверия, что у PostgREST/RLS, НЕ требует живой auth-сессии (переживает
  single-session-per-user, когда старая вкладка получает
  `session_not_found`); fallback — `GET /auth/v1/user`.
- Логика: unknown/free course → 400; активная строка с живой Stripe-подпиской
  → 409 `already_active` (истёкшая/отменённая может купить заново); customer
  создаётся лениво и запоминается в `profiles.stripe_customer_id`; Checkout
  Session `mode:"subscription"` с `price_data` (не заранее созданный Price);
  metadata `{user_id, course_id}` кладётся и на сессию, и на подписку
  (`subscription_data.metadata`), чтобы вебхук всегда мог смапить событие;
  success/cancel URL → `app.html?checkout=…`. Функция НИКОГДА не пишет
  `user_courses`.

## stripe-webhook (`.../stripe-webhook/index.ts`, 132 строки)

- Deploy с `--no-verify-jwt` (Stripe не шлёт Supabase JWT); подпись события
  проверяется `constructEventAsync` + `STRIPE_WEBHOOK_SECRET`; плохая подпись
  → 400.
- Единственный писатель платных прав. `GRACE_MS = 2 суток` поверх конца
  оплаченного периода — медленный вебхук/ретрай карты не «моргает» доступом;
  дневной cron гасит строку тоже только после этого буфера.
- События: `checkout.session.completed` (mode subscription → retrieve
  подписки; если на подписке нет metadata — докладывает из session,
  belt-and-braces для ручных тестов из дашборда) и `invoice.paid` → upsert
  `user_courses` (active, expires_at = период+grace, auto_renew =
  `!cancel_at_period_end`, stripe_subscription_id);
  `customer.subscription.updated` → только `auto_renew` (доступ не трогается
  — «пользователь сохраняет то, за что заплатил»);
  `customer.subscription.deleted` → status inactive, auto_renew false,
  expires_at = сейчас. Прочие события — acknowledged.
- Ошибка обработчика → 500, чтобы Stripe ретраил (записи идемпотентны).
- `periodEndMs()` читает `current_period_end` и с подписки, и с её item
  (разные версии Stripe API); fallback — now + 31 день.

## stripe-portal (`.../stripe-portal/index.ts`, 90 строк)

Возвращает URL Stripe Customer Portal (отмена → `cancel_at_period_end`, смена
карты, инвойсы). Та же JWKS-проверка вызывающего, что в checkout. Требует
одноразовой активации портала в Stripe Dashboard (комментарий в файле) —
сделана ли она, из репозитория не видно (UNKNOWN).

## ticket-email (`.../ticket-email/index.ts`, 126 строк)

- Авторизация: заголовок `x-ticket-secret` = `TICKET_WEBHOOK_SECRET`.
- Вызывается pg_net-триггером (`supabase/ticket_email_trigger.sql`) на INSERT
  и на UPDATE со сменой статуса на done/rejected.
- Словарь `COPY`: EN/ES/RU × стадии received/done/rejected × kind
  request/complaint (+question для received) — subject и body каждой
  комбинации зашиты в файле; отправка через Resend, from
  `LICENA <noreply@licena.us>`; в письмо включается исходное сообщение
  пользователя и `admin_note` (в done/rejected).

## ticket-issue (`.../ticket-issue/index.ts`, 103 строки)

- Авторизация `x-ticket-secret`; вызывается триггером только для kind
  `request|complaint` при INSERT.
- Создаёт GitHub issue в `lalianamen/llicena` (REST API, fine-grained PAT
  `GH_ISSUE_TOKEN` с правом Issues RW): title `[chat] <kind>: <70 симв.>`,
  body с цитатой сообщения, ticket id, email, locale, course_id,
  target_lang.
- Greenlight-политика (комментарий в коде): label ставится только `support`;
  label `from-chat`, запускающий workflow автоматизации, владелец добавляет
  вручную — потому что публичный чат-эндпоинт не rate-limited и полная
  автоматизация была бы вектором abuse/расходов.
- Дополнительно шлёт владельцу письмо-уведомление через Resend
  (`OWNER_EMAIL`, дефолт — адрес проекта в коде).

## ticket-status (`.../ticket-status/index.ts`, 57 строк)

- Авторизация `x-ticket-secret`. Вход `{ticket_id, status}` (status из
  `new|in_progress|done|rejected`, дефолт `done`).
- Один привилегированный UPDATE `support_tickets` service role'ом (RLS не
  даёт публичному ключу менять статусы); смена статуса сама триггерит
  письмо пользователю. Вызывается workflow'ом `claude-ticket-resolved` при
  merge PR.

## daily-stats (`.../daily-stats/index.ts`, 265 строк)

- Ежедневный отчёт о визитах на email (owner request 2026-07-17); Verify JWT
  ON — cron шлёт service-role JWT (`supabase/sql/cron-daily-stats.sql`).
- Читает `page_views` service role'ом; агрегация по дням
  `America/Los_Angeles`.
- Классификация каналов: `?src=`-метка приоритетнее реферера (карта `CH_TAG`:
  fb/facebook→facebook, ig/insta→instagram, tg→telegram, tt→tiktok,
  flyer/qr→flyer; неизвестная метка→other), затем хост реферера; null =
  внутренняя навигация.
- Разбивка по каналам (`CH_ROWS`): с 2026-08-30 добавлены строки «Поиск
  (Google и др.)» и «AI-ассистенты», а `other` переименован из «Прочее (поиск
  и др.)» в «Прочее» — прежняя подпись перестала быть верной. Требует
  переразвёртывания функции.
- Состав письма: вчерашние числа, разбивка по источникам, 7-дневная таблица,
  топ страниц; 1-го числа месяца — дополнительно месячный rollup (границу
  месяца определяет сама ежедневная джоба). KPI-функции БД `daily_signups`/
  `recent_signups` — атрибуция регистраций к каналам.
- Отправка Resend; получатели из `STATS_EMAIL` (comma-separated; дефолт в
  коде — личный адрес получателя отчёта, в базу знаний не переносится).

## marketing-aggregates (`.../marketing-aggregates/index.ts` 169 строк + `core.ts` 252 строки)

Добавлено 2026-08-30. Задеплоена на живой проект владельцем 2026-08-30
(подтверждение: `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`);
исходники в `main` @ `ae13124` (мерж 2026-08-30; разработка велась на ветке
`claude/marketing-analytics-aggregates`).

- Назначение: пишет обезличенные ежедневные агрегаты в `marketing_daily_metrics`,
  `marketing_channel_daily`, `marketing_page_daily` и снепшот текущего состояния
  в `marketing_state_snapshots`. Отдельная от `daily-stats` функция сознательно:
  сбой агрегатов не может стоить владельцу ежедневного письма. Единственное
  общее с письмом — блок таксономии каналов, который обязан оставаться
  побайтово одинаковым в обоих файлах; правка в нём меняет и письмо
  (2026-08-30 `daily-stats/index.ts` был изменён именно так — см. ниже).
- Разделение файлов: `core.ts` — чистая логика без Deno и сети (импортируется
  тестами `scripts/test-marketing-core.mjs` через `node --experimental-strip-types`,
  то есть тестируется production-код, а не копия); `index.ts` — только I/O:
  авторизация, пагинация, запись.
- Авторизация вызывающего: `verify_jwt` пропускает ЛЮБОЙ проектный JWT, включая
  публичный anon-ключ, поэтому функция дополнительно требует сервисные
  credentials — прямое совпадение с `SUPABASE_SERVICE_ROLE_KEY` либо, для
  нового формата `sb_secret_...`, проверка возможностей через
  `/auth/v1/admin/users` (200 отвечают только сервисные ключи). Иначе 403.
  Тот же паттерн, что у `gsc-sync`.
- Вход: `{"days": N}`, по умолчанию 3, clamp 1–40 (bounded backfill).
  Целевые дни — последние N **завершённых** календарных дней Pacific,
  вычисленные календарной арифметикой от текущей PT-даты (`core.targetDaysBack`);
  текущий неполный день никогда не входит.
- Чтение `page_views` постраничное (PAGE_SIZE 1000, MAX_ROWS 400000) и
  **fail-closed**: при исчерпании бюджета возвращается HTTP 507
  `source_window_truncated` и не пишется ничего.
- Детерминизм: атрибуция канала устройства считается по ФИКСИРОВАННОМУ окну
  28 дней, заканчивающемуся концом целевого дня (`ATTRIBUTION_LOOKBACK_DAYS`),
  а не по ширине backfill — день, посчитанный отдельно, равен тому же дню внутри
  любого более широкого прогона.
- Запись: по одному дню за раз через RPC `marketing_replace_day` (delete+insert
  в одной транзакции). Снепшот текущего состояния — только за день запуска.
- Таксономия каналов — побайтовая копия блока из `daily-stats/index.ts` между
  маркерами `channel-taxonomy-begin/end`; расхождение ловит
  `scripts/check-channel-parity.js`. С 2026-08-30 (`c2e9dd6`, решение
  владельца) из бакета `other` выделены `search` (google, bing, duckduckgo,
  yandex, ya.ru, search.yahoo) и `ai` (chatgpt, openai, perplexity, claude.ai,
  gemini.google, copilot); проверка `ai` идёт раньше `search`, чтобы
  `gemini.google.com` не считался поиском, а поддомены `mail.`/`docs.`/`drive.`
  остаются в `other` как ссылки из приложений, а не поисковый трафик.
- Тестеры (`profiles.is_tester`) исключены из всех пользовательских метрик;
  устройства до входа отфильтровать нельзя — то же ограничение, что у письма.
- Секреты: сверх стандартных `SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` — нет.
- Расписание: cron-задание `marketing-aggregates`, `30 15 * * *`
  (`supabase/sql/cron-marketing-aggregates.sql`).

## gsc-sync (`.../gsc-sync/index.ts`, 373 строк)

Добавлена 2026-08-25 (задача владельца). Синхронизация Google Search Console →
`public.gsc_snapshots`. POST-only; помимо `verify_jwt` требует в Authorization
сам service-role ключ (anon-JWT отклоняется, стадия `auth`, 401 — фикс по
результату адверсариальной ревизии).

- Свойство: `https://licena.us/` (URL-encoded в пути API). Скоуп только
  `https://www.googleapis.com/auth/webmasters.readonly`.
- Аутентификация Google: RS256 JWT (WebCrypto, импорт PKCS8 из секрета
  `GSC_SERVICE_ACCOUNT_JSON`; `iat` со сдвигом −60 с), обмен на access token
  по `token_uri` из JSON (по умолчанию `https://oauth2.googleapis.com/token`).
  Секрет читается только в память; в логи/ответы не попадает.
- Окно: последние 28 полных дней, конец — 3 дня назад; календарные дни
  считаются в `America/Los_Angeles` (GSC отдаёт даты в Pacific). Опциональное
  тело `{"days": 1..90}` меняет длину окна.
- Пять датасетов: query+page, query+country, query+device, page+device, date;
  `rowLimit` 25000, пагинация по `startRow`, кап 4 страницы на датасет с
  флагом `truncated`; `dataState:"final"`, `type:"web"`.
- Схема `gsc_snapshots` не хранится в репо (таблица создана в Supabase
  напрямую) — функция перед записью читает её фактическую схему через
  OpenAPI-описание PostgREST (service role) и адаптирует вставку: карта
  алиасов колонок (fetched_at/property/start_date/end_date/dimensions/
  row_count/payload), `dimensions` в форму text[]/text/jsonb по типу колонки,
  `payload` сериализуется строкой для text-колонки; незаполнимые NOT NULL
  колонки — быстрый отказ (стадия `schema`).
- Запись: по одной строке снапшота на датасет (5 отдельных insert), только
  service role; политики RLS не создаются и не меняются.
- Ошибки типизированы по стадиям без секретов: `auth`, `secret`,
  `google-oauth`, `gsc-permission`, `gsc-property`, `gsc-quota` (включая 403
  с reason rateLimit/quota), `gsc-query`, `schema`, `db-insert` (с числом уже
  вставленных). Успех: `{ok, property, start_date, end_date, snapshots, rows,
  datasets[], columns_used}`.
- Деплой: выполнен владельцем 2026-08-25 через Dashboard (Via Editor).
  Итоговая версия кода: `384526c` (после `f1390ad` добавлены: допуск обоих
  поколений API-ключей через capability-проверку GoTrue admin-эндпоинтом,
  самодиагностика auth-отказа по role/длинам, исключение identity `id` из
  предзаписной проверки required-колонок).
- Первый боевой запуск 2026-08-25 подтверждён ответом функции: окно
  2026-07-26…2026-08-22 (28 дней), 5 снапшотов, 324 строки GSC
  (query+page 77, query+country 87, query+device 79, page+device 53,
  date 28), без пагинации и усечения; фактические колонки:
  property → `site_url`, остальные — одноимённые.

## Секреты (имена; значения только в Supabase → Edge Functions → Secrets)

`CLAUDE_API_KEY` (assistant); `STRIPE_SECRET_KEY` (все stripe-*);
`STRIPE_WEBHOOK_SECRET` (webhook); `RESEND_API_KEY` (ticket-email,
ticket-issue, daily-stats); `TICKET_WEBHOOK_SECRET` (ticket-*);
`GH_ISSUE_TOKEN`, `OWNER_EMAIL` (ticket-issue); `STATS_EMAIL` (daily-stats);
`SUPABASE_URL`/`SUPABASE_ANON_KEY`/`SUPABASE_SERVICE_ROLE_KEY` — инжектируются
платформой. Verify JWT: OFF только у `stripe-webhook`.
- `GSC_SERVICE_ACCOUNT_JSON` — полный JSON сервисного аккаунта Google (gsc-sync; доступ Restricted к Search Console property `https://licena.us/`).

## Source References

- `supabase/functions/marketing-aggregates/index.ts` и `core.ts`,
  `supabase/sql/marketing-daily-aggregates.sql`,
  `supabase/sql/cron-marketing-aggregates.sql`,
  `scripts/test-marketing-core.mjs`, `scripts/check-channel-parity.js`
  (`main` @ `ae13124`; добавлено 2026-08-30)
- `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md` (факт деплоя)
- `supabase/functions/gsc-sync/index.ts` (добавлено 2026-08-25)

- `supabase/functions/assistant/index.ts`, `stripe-checkout/index.ts`,
  `stripe-webhook/index.ts`, `stripe-portal/index.ts`,
  `ticket-email/index.ts`, `ticket-issue/index.ts`, `ticket-status/index.ts`,
  `daily-stats/index.ts` — прочитаны полностью 2026-08-05
- `supabase/ticket_email_trigger.sql`, `ticket_issue_trigger.sql`,
  `supabase/sql/cron-daily-stats.sql`, `report-kpi.sql` (вызывающая сторона)
- `js/support.js`, `js/app-cabinet.js` (клиентские вызовы)

## Verification Status

**Verified** — все 9 файлов функций прочитаны полностью; каждое утверждение
прослеживается к строкам кода. UNKNOWN: фактические настройки в дашбордах
(Verify JWT-тумблеры, активация Customer Portal, набор событий в Stripe
Webhooks, значения секретов) — из репозитория не видны; код фиксирует
требуемые настройки в комментариях.
