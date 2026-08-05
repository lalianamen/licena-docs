# 18 — Зависимости

Последняя сверка: 2026-08-05
Правило: для каждой зависимости — версия, назначение, место использования и
ИСТОЧНИК версии. Суждений об устарелости/уязвимости нет (внешняя проверка не
проводилась). В репозитории НЕТ `package.json`, лок-файлов и `.nvmrc`
(`find` 2026-08-05 — 0 результатов): единственная клиентская библиотека
самохостится файлом, серверные зависимости объявлены в import-спецификаторах
Deno.

## Клиентские (браузер)

| Зависимость | Версия | Назначение | Где | Источник версии |
|---|---|---|---|---|
| `@supabase/supabase-js` | **2.110.0** | Auth + PostgREST + Functions-клиент | `js/vendor/supabase-js-2.110.0.js` (204 КБ), подключён в `index/app/course.html` | имя vendor-файла (точная копия, без CDN) |
| Google Fonts: Archivo (wght 500–900), IBM Plex Mono (400–600), IBM Plex Sans (400–600) | n/a (hosted-сервис) | шрифты, `display=swap` | `index.html:32–34` (`<link>` + preconnect); CSP разрешает только fonts.googleapis/gstatic | URL запроса css2 |

Прочего клиентского вендора нет: остальной JS — собственный код проекта.

## Серверные (Supabase Edge Functions, Deno)

| Импорт | Версия | Назначение | Функции | Источник версии |
|---|---|---|---|---|
| `npm:stripe` | **@17** (мажор) | Stripe SDK; `apiVersion` в коде НЕ задан → используется дефолт SDK | stripe-checkout, stripe-webhook, stripe-portal | import-спецификатор |
| `npm:@supabase/supabase-js` | **@2** (мажор) | service-role клиент БД | 6 функций (все, кроме ticket-email/ticket-issue) | import-спецификатор |
| `npm:jose` | **@5** (мажор) | JWKS-проверка JWT | stripe-checkout, stripe-portal | import-спецификатор |
| `npm:@anthropic-ai/sdk` | **не зафиксирована** (без версии в импорте) | Claude API | assistant | import-спецификатор без версии |
| Deno runtime | **UNKNOWN** (управляется Supabase) | среда исполнения функций | все 8 | — |

## Инструментальные

| Зависимость | Версия | Назначение | Источник версии |
|---|---|---|---|
| Node.js | **не зафиксирована** (нет `.nvmrc`/`engines`) | `scripts/*.js` (только встроенные модули fs/path/vm/child_process) | — |
| `actions/checkout` | **@v4** | клонирование в 4 workflow | `.github/workflows/*.yml` |
| `anthropics/claude-code-action` | **@v1** | запуск Claude Code по тикету | `.github/workflows/claude-support.yml` |
| Python 3 (`http.server`) | не зафиксирована | локальный запуск статики (опционально) | `README.md` осн. репо |

## Внешние сервисы (версии API)

| Сервис | Версия/идентификатор | Назначение | Где |
|---|---|---|---|
| GitHub Pages | n/a | хостинг статики с `main` | `CNAME`, `.nojekyll`, README |
| Supabase | проект `vewhmndummfhnbxnrqya.supabase.co`; план/регион UNKNOWN | БД, Auth, Functions, Storage, pg_cron/pg_net | `js/supabase-client.js` |
| Stripe API | версия не пиннится (SDK-дефолт stripe@17) | подписки $20/мес | три stripe-функции |
| Anthropic API | модель **`claude-sonnet-4-6`** | AI-саппорт | `assistant/index.ts:18` |
| Anthropic API (CI) | модель UNKNOWN (выбирает claude-code-action) | работа тикетов | `claude-support.yml` |
| Resend | HTTP API, версия не пиннится | письма от `noreply@licena.us` | ticket-email, ticket-issue, daily-stats |
| Telegram Bot API | версия не пиннится | автопост в `@licena_us` | `telegram-post.yml` |
| Facebook Graph API | **v23.0** (в URL) | автопост на страницу | `facebook-post.yml:71` |
| Google Search Console | n/a | поисковая аналитика | `googlec4665304f2eceeb0.html` |

## Зависимости без зафиксированной версии (сводно)

`@anthropic-ai/sdk` (импорт без версии); Stripe `apiVersion` (SDK-дефолт);
Deno runtime; Node.js; Resend API; Telegram Bot API; Google Fonts; модель
Claude в claude-code-action. Мажор-только пины: `stripe@17`,
`supabase-js@2` (server), `jose@5`, `checkout@v4`, `claude-code-action@v1`.

## Source References

- `js/vendor/supabase-js-2.110.0.js` (имя файла), `index.html:32–34`,
  `supabase/functions/*/index.ts` (import-спецификаторы, grep 2026-08-05),
  `.github/workflows/*.yml`, `scripts/*.js` (require-список),
  `README.md` осн. репо, `js/supabase-client.js`,
  `assistant/index.ts:18`, `facebook-post.yml:71`
- `find package.json|*.lock|.nvmrc` — 0 результатов (2026-08-05)

## Verification Status

**Verified** — каждая строка таблиц взята из названного файла; UNKNOWN
отмечает отсутствующие фиксации версий. Внешняя проверка
актуальности/уязвимостей не проводилась и не утверждается.
