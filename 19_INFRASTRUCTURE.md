# 19 — Инфраструктура

Последняя сверка: 2026-08-05

Дополнение 2026-08-12: внешний мониторинг доступности — UptimeRobot
(бесплатный тариф, аккаунт владельца): «LICENA site» (HTTP/S,
https://licena.us, 5 мин) и «LICENA Supabase» (Keyword-монитор: GET
`<project>.supabase.co/auth/v1/health?apikey=<публичный sb_publishable-ключ
из js/supabase-client.js>`, keyword `GoTrue`, инцидент при отсутствии);
email-оповещения владельцу. Keyword-тип выбран потому, что HTTP/S-монитор
бесплатного тарифа шлёт только HEAD, а health-эндпоинт принимает лишь GET
(405) и требует apikey (иначе 401).
Только данные, доступные из репозитория `lalianamen/llicena`; все настройки
дашбордов и DNS-записи вне репо — UNKNOWN.

## Хостинг фронтенда — GitHub Pages

- Отдача ветки `main` из корня как есть; деплой = push в `main`
  (`README.md` осн. репо, раздел Deploy; `CLAUDE.md`: «GitHub Pages serves
  main verbatim»).
- Кастомный домен: `licena.us` (файл `CNAME`); `.nojekyll` отключает
  Jekyll-обработку. Регистратор, DNS-записи (A/AAAA/CNAME, SPF/DKIM/DMARC),
  enforce-HTTPS-тумблер — UNKNOWN (вне репозитория).
- Vercel/Netlify-конфигураций в репо нет; MCP-инструменты Vercel к проекту
  не привязывались (в этой сверке не проверялось и не утверждается).

## Git

- Репозитории: `lalianamen/llicena` (код), `lalianamen/licena-docs` (эта
  база знаний, публичный). Ветки llicena: `main` (прод),
  `content-banks-src` (исходники платных банков); прочие — UNKNOWN
  (shallow-клон). Защита веток, обязательные ревью — UNKNOWN.
- Требование workflow: установлено GitHub App «Claude»
  (`claude-support.yml`, комментарий Setup).

## Supabase (проект `vewhmndummfhnbxnrqya.supabase.co`)

| Компонент | Состав (по репо) |
|---|---|
| Postgres | 9 таблиц + RLS, 7 функций, 3 триггера (`04_DATABASE.md`) |
| Расширения | `pg_cron`, `pg_net` (включаются скриптами) |
| Cron | `expire-subscriptions` 30 7 * * * ; `daily-stats` 0 15 * * * (файл рекомендует Dashboard→Cron) |
| Auth | email+пароль, подтверждение, сброс; implicit flow на клиенте; серверные настройки — UNKNOWN |
| Edge Functions | 8 (`06_FUNCTIONS.md`); деплой по комментариям в файлах: `supabase functions deploy <name>`, для webhook — `--no-verify-jwt` |
| Storage | приватный бакет `support-uploads` (≤2 МБ, image/*) |
| Secrets | имена: CLAUDE_API_KEY, STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, RESEND_API_KEY, TICKET_WEBHOOK_SECRET, GH_ISSUE_TOKEN, OWNER_EMAIL, STATS_EMAIL (+платформенные SUPABASE_*) |

План/регион/квоты проекта, фактическое состояние cron и применённость
SQL-скриптов — UNKNOWN.

## Stripe

Аккаунт ACTIVE на 2026-07-28 (go/no-go в шапке `stripe-payments.sql`);
endpoint вебхука `…/functions/v1/stripe-webhook`, ожидаемые события:
checkout.session.completed, invoice.paid, customer.subscription.updated,
customer.subscription.deleted (комментарий `stripe-webhook/index.ts`);
требуется одноразовая активация Customer Portal (комментарий
`stripe-portal/index.ts`). Live/test-режим, фактическая настройка событий,
налоги — UNKNOWN.

## Email

Транзакционные письма — Resend, отправитель `LICENA <noreply@licena.us>`
(ticket-email, ticket-issue, daily-stats). Верификация домена в Resend,
SPF/DKIM-записи — UNKNOWN. Шаблоны писем Supabase Auth —
`supabase/email-templates/confirm-signup.html`, `reset-password.html`
(+ копия в `docs/email/`).

## GitHub Actions (4 workflow, прочитаны полностью)

| Workflow | Триггер | Что делает | Секреты (имена) |
|---|---|---|---|
| `claude-support.yml` | issues opened/labeled, гейт: label `from-chat` | `anthropics/claude-code-action@v1`: работает тикет по CLAUDE.md, ветка `claude/ticket-<slug>`, `--max-turns 50`, `--permission-mode acceptEdits`, allowed tools Bash/Edit/Write/Read/Grep/Glob/WebFetch/WebSearch; открывает PR, НИКОГДА не мержит; permissions: contents/issues/pull-requests write + id-token | `ANTHROPIC_API_KEY` |
| `claude-ticket-resolved.yml` | pull_request closed; гейт: merged == true и ветка `claude/ticket-*` | достаёт номер issue из PR-body (`#N`), UUID тикета из issue-body, POST в `ticket-status` (`done`) → триггер шлёт пользователю письмо; без секрета — безопасный no-op | `TICKET_WEBHOOK_SECRET` (+ `github.token`) |
| `telegram-post.yml` | push в main файлов `docs/smm/queue/*.txt` (только добавленных, `--diff-filter=A`); + workflow_dispatch с именем файла | `sendMessage` в канал `@licena_us` текстом файла; concurrency group без отмены | `TELEGRAM_BOT_TOKEN` |
| `facebook-post.yml` | push в main `docs/smm/queue-fb/*.txt`; + dispatch | Graph API v23.0 `POST /me/feed`; последняя https-ссылка файла прикладывается как `link` | `FB_PAGE_TOKEN` |

CI-проверок кода (запуск `verify.js`/тестов на push/PR) среди workflow НЕТ
(`20_VERIFICATION.md`).

## Социальные площадки (идентификаторы из репо)

Telegram `@licena_us` (workflow + seo.js sameAs); Facebook page id
`61592095073500`; Instagram `@licena_us`; TikTok `@licena_us`
(`js/seo.js:34–39`). Доступы/владение аккаунтами — вне репо.

## Мониторинг и алертинг

Специальных систем в репо нет. Единственная регулярная отчётность —
письмо `daily-stats`; уведомления владельцу о тикетах — письма
`ticket-issue`. Логи функций — `console.error` в код (просмотр — дашборд
Supabase, UNKNOWN).

## Локальная разработка

Статика открывается напрямую или `python3 -m http.server 8000`
(`README.md` осн. репо). Секретов для локального фронтенда не требуется
(publishable-ключ в коде).

## UNKNOWN (сводно)

DNS/регистратор/SPF-DKIM; настройки GitHub Pages (HTTPS) и защиты веток;
план/регион/квоты Supabase; фактическое состояние cron и применённость SQL;
live/test Stripe и настройка вебхука/портала; верификация домена в Resend;
владение соц-аккаунтами; модель Claude в claude-code-action.

## Source References

- `CNAME`, `.nojekyll`, `README.md` и `CLAUDE.md` осн. репо
- `.github/workflows/*.yml` — все 4 полностью
- `js/supabase-client.js`, `supabase/functions/*/index.ts` (деплой-комментарии,
  секреты), `supabase/sql/cron-daily-stats.sql`, `subscriptions-schema.sql`,
  `stripe-payments.sql` (go/no-go), `stripe-portal/index.ts`,
  `stripe-webhook/index.ts`, `supabase/support_uploads_storage.sql`,
  `supabase/email-templates/`, `js/seo.js:29–39`
- git: `git branch -r` (shallow)

## Verification Status

**Partially Verified** — всё перечисленное подтверждено файлами репозитория;
раздел UNKNOWN покрывает недоступные из репо настройки внешних систем.
