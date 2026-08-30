# 08 — Безопасность

Последняя сверка: 2026-08-05 · 2026-08-30 (точечная: маркетинговые агрегаты)
Только факты из кода и внутренних документов основного репозитория; без
оценок и рекомендаций. Слабые места перечислены исключительно там, где они
прямо названы в коде/документах проекта или следуют из прочитанного кода.

## Действующие механизмы (по коду)

| Механизм | Реализация | Источник |
|---|---|---|
| RLS на всех таблицах | owner-scoped политики; `bank_questions` — чтение только владельцу активного курса; `page_views` insert-only; `reviews` — читаются только approved; серверные таблицы без политик | `docs/schema.sql`, `supabase/sql/*.sql` (полный разбор — `04_DATABASE.md`) |
| Платные права пишутся только сервером | RLS free-tier-only + `stripe-webhook` (service role) + `start_trial` (security definer) | `supabase/sql/stripe-payments.sql` SECTION B; `stripe-webhook/index.ts`; `trial-3day.sql` |
| Платный контент не в статике | 12+ платных банков удалены из `main` 2026-07-29; отдаются из `bank_questions` под RLS; анонимный запрос получает пустой ответ | `CLAUDE.md` осн. репо; `js/app-course.js:48–56` |
| CSP | `<meta http-equiv="Content-Security-Policy">`: `default-src 'self'; script-src 'self'; …; connect-src 'self' https://*.supabase.co wss://*.supabase.co; object-src 'none'; base-uri 'self'; form-action 'self'` — на `index.html:6`, `app.html:7`, `course.html:7`, practice-, guides-, about-страницах. Inline-скрипты запрещены (`js/pwa.js`, комментарий) | сами HTML-файлы |
| Без CDN-зависимостей | supabase-js самохостится: `js/vendor/supabase-js-2.110.0.js` (точная версия); внешних `<script src>` нет | листинги `<script>` в HTML |
| Верификация Stripe-вебхука | подпись `stripe-signature` + `STRIPE_WEBHOOK_SECRET`, плохая подпись → 400 | `stripe-webhook/index.ts` |
| JWT по подписи (JWKS) | Stripe-функции проверяют токен криптографически, как PostgREST | `stripe-checkout/index.ts:60–91` |
| Приватные вебхуки БД | заголовок `x-ticket-secret`; publishable key в Authorization — только для Edge-гейтвея | `supabase/ticket_*_trigger.sql` |
| Скриншоты саппорта | приватный бакет, ≤ 2 МБ, только jpeg/png/webp, RLS «только своя папка», доступ по временным signed URL; функция принимает только URL своего Supabase-хоста | `supabase/support_uploads_storage.sql`; `assistant/index.ts` (`isShotUrl`) |
| Анти-перечисление аккаунтов | «email уже занят» неотличим от новой регистрации | `js/app.js:206–210` |
| XSS-дисциплина | пользовательские значения — `textContent`; контент вопросов — `esc()` перед `innerHTML`; sink'ов не найдено (аудит 2026-06-24) | `docs/security-todo.md`, «What's already solid» |
| Секреты вне репозитория | `.gitignore`: `.env*`, seed-файлы; service_role в репо и git-истории отсутствует (проверка 2026-06-24); секреты функций — только имена в коде | `.gitignore`; `docs/security-todo.md` |
| Анти-шеринг | серверный `register_device()`: 5 устройств, swap ≤ 1/30 дней | `supabase/devices_anti_sharing.sql` (детали — `07_AUTH_ACCESS.md`) |
| Модерация UGC | отзывы публикуются только владельцем (insert принудительно `pending`); тикеты меняют статус только service role | `supabase/reviews.sql`, `support_tickets.sql` |
| AI-ассистент | ключ API на сервере; лимиты 6/40/8000; промпт-инструкция игнорировать смену роли из сообщений/скриншотов/поиска; принимаются только свои storage-URL | `assistant/index.ts` |
| Защита от trial-фарминга | `course_trials` не очищается; одна выдача на (user, course) навсегда | `trial-3day.sql` |
| Crawl-щиты | `robots.txt` Disallow: `/supabase/`, `/docs/`, `/js/questions/`, `/js/guides/` — сам файл помечает это как «crawl-level deterrent only» | `robots.txt` |

## Дополнение 2026-08-30: default privileges Supabase и `TRUNCATE` в обход RLS

Установленный факт, зафиксированный при применении миграции
`supabase/sql/marketing-daily-aggregates.sql` на живой базе (выгрузка
`information_schema.role_table_grants`, 2026-08-30, отчёт
`tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`):

- Default privileges Supabase в схеме `public` выдают ролям `anon` и
  `authenticated` **все** привилегии (SELECT, INSERT, UPDATE, DELETE,
  TRUNCATE, REFERENCES, TRIGGER) на каждую **новую** таблицу — независимо от
  того, что записано в миграции.
- Включённый RLS без политик закрывает доступ к строкам (deny-all для ролей
  без BYPASSRLS), поэтому чтения и записи строк такой грант не даёт.
- **`TRUNCATE` не подчиняется RLS.** Право `TRUNCATE` у клиентской роли
  означает возможность очистить таблицу целиком, несмотря на RLS.
- Следствие для этой миграции: в неё добавлен явный
  `revoke all on <таблица> from anon, authenticated` (коммит `ae13124`
  ветки `claude/marketing-analytics-aggregates`); та же команда выполнена на
  живой базе. Повторная проверка грантов ПОСЛЕ `revoke` — `UNKNOWN`
  (контрольный запрос не выполнялся, см. отчёт).
- Область факта: проверялись только 4 таблицы `marketing_*`. Состояние
  грантов на ранее созданных таблицах в рамках этой сверки **не**
  проверялось — `UNKNOWN`.

Гейт авторизации Edge Function `marketing-aggregates`: `verify_jwt`
пропускает любой валидный проектный JWT, включая публичный anon-ключ,
поэтому функция дополнительно требует сервисные credentials (прямое
совпадение с `SUPABASE_SERVICE_ROLE_KEY` либо проверка возможностей через
`/auth/v1/admin/users` для ключей формата `sb_secret_...`), иначе 403 — тот
же паттерн, что у `gsc-sync` (`supabase/functions/marketing-aggregates/index.ts`).

Хранение ключа в cron: service-role-ключ записан в теле cron-задачи в
`cron.job` (`supabase/sql/cron-marketing-aggregates.sql`) — тот же подход,
что и у `supabase/sql/cron-daily-stats.sql`.

Security Advisor Supabase на 2026-08-30: 0 errors, 22 warnings, 11
suggestions; ни одно предупреждение не относится к объектам `marketing_*`
(полная выгрузка предоставлена владельцем). Содержание остальных 22
предупреждений в этой сверке не разбиралось — `UNKNOWN`.

## Задокументированные в самом проекте слабые места (факты, не аудит)

- **Файлы под `Disallow` остаются публично доступными по URL** — `robots.txt`
  прямо говорит: «these files stay public on the host»; реальная защита
  платного контента — БД (см. выше), но бесплатные банки и внутренние
  `docs/**` основного репо доступны любому, кто знает URL (осознанное
  решение: free banks — lead magnet, `CLAUDE.md` осн. репо).
- **`profiles.is_tester` можно выставить себе** API-вызовом (own-update
  политика); в `tester-account.sql` это описано с обоснованием: открывается
  только карточка тест-курса, вопросы остаются за RLS — «flag is a curtain,
  not a lock».
- **Публичный чат-эндпоинт не rate-limited** — комментарий в
  `ticket-issue/index.ts` называет это причиной greenlight-политики (issue
  не запускает автоматизацию без ручного label): полная автоматизация была
  бы «abuse/cost vector (anyone could spam paid Claude runs)».
- **CSP-мета отсутствует на `privacy.html` и `terms.html`** (grep по
  `Content-Security-Policy`, 2026-08-05); на остальных страницах есть.
- **`x-ticket-secret` в SQL-файлах — плейсхолдер** `__TICKET_WEBHOOK_SECRET__`,
  заменяется владельцем при запуске (сам секрет в репо не хранится) —
  `ticket_*_trigger.sql`.

## Статус пунктов `docs/security-todo.md` (аудит владельца от 2026-06-24) на 2026-08-05

Фактическое состояние каждого пункта по текущему коду:

| № | Пункт (2026-06-24) | Состояние на 2026-08-05 (по коду) |
|---|---|---|
| 1 | Клиент может сам выдать себе любой курс | Закрыт в коде: RLS free-tier-only + вебхук + `start_trial` (`stripe-payments.sql` SECTION B, 2026-08-01). Применение к живой БД — UNKNOWN |
| 2 | Банки — публичные статические файлы | Закрыт для платных: перенос в `bank_questions` (RLS), статика удалена из `main` 2026-07-29. Бесплатные — публичны намеренно |
| 3 | Нет CSP, CDN-скрипт без SRI | Закрыт: CSP-мета на страницах; supabase-js самохостится точной версией (`js/vendor/supabase-js-2.110.0.js`), CDN-подключения нет |
| 4 | `courseId` из URL → путь скрипта | Изменено: платные банки не грузятся статикой вовсе; для бесплатных `loadScript("js/questions/"+id+".js")` — сохранность точечной валидации id в текущем коде не проверялась в этой сверке (UNKNOWN) |
| 5 | Нет клиентской парольной политики | Закрыт частично: `passOk()` требует заглавную и символ (`js/app.js:34`); серверный минимум — UNKNOWN |
| 6 | Docs описывают непостроенный анти-шеринг | Закрыт: `register_device()` реализован (2026-07-20); `docs/access-control.md` сам помечает старую модель как «future design» |
| 7 | В README устаревший демо-логин | Закрыт: в текущем `README.md` демо-креденшелов нет (прочитан полностью) |
| 8 | Нет `.gitignore` | Закрыт: `.gitignore` существует, покрывает `.env*`, seed-файлы |
| 9 | Остался тестовый пользователь в auth.users | UNKNOWN: живая БД из репозитория не видна |

## Что невозможно проверить только по репозиторию (UNKNOWN)

- Какие SQL-скрипты фактически выполнены в живой БД (включая SECTION B) и
  текущие политики/расширения там.
- Настройки Supabase: Auth (мин. пароль, single-session, TTL токенов),
  Verify-JWT-тумблеры функций, секреты, состояние cron.
- Настройки Stripe Dashboard: события вебхука, активация Customer Portal,
  налоги/квитанции.
- Утечки/инциденты, логи, фактический трафик злоупотреблений.
- Права доступа к GitHub-репозиториям и защита веток.

## Source References

- `docs/security-todo.md`, `docs/access-control.md` (прочитаны полностью)
- `robots.txt`, `.gitignore`, CSP-мета: `index.html:6`, `app.html:7`,
  `course.html:7`, `practice/c-10-electrical/index.html`,
  `guides/c-10-electrical-license-california/index.html`, `about/index.html`;
  отсутствие в `privacy.html`, `terms.html` — grep 2026-08-05
- `supabase/sql/stripe-payments.sql`, `bank-questions-schema.sql`,
  `trial-3day.sql`, `tester-account.sql`, `devices_anti_sharing.sql`,
  `reviews.sql`, `support_tickets.sql`, `support_uploads_storage.sql`,
  `ticket_email_trigger.sql`, `ticket_issue_trigger.sql`
- `supabase/functions/*` (все 8, полностью), `js/app.js`, `js/app-course.js`,
  `js/supabase-client.js`, `js/vendor/supabase-js-2.110.0.js` (наличие),
  `CLAUDE.md` осн. репо

## Verification Status

**Partially Verified.**

- Проверено чтением кода: все строки таблицы «Действующие механизмы» и
  раздела «слабые места», статусы пунктов 2, 3, 5 (клиентская часть), 6, 7, 8.
- Взято из `docs/security-todo.md` без независимой перепроверки: пункты
  «verified live» от 2026-06-24 (RLS-ответы анонима, отсутствие ключа в
  git-истории, mailer_autoconfirm, XSS-обход).
- UNKNOWN-позиции перечислены в отдельном разделе; плюс п.4 и п.9 таблицы
  статусов.
