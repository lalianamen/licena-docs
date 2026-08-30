# TASK — Marketing Analytics Daily Aggregates

Последняя сверка: 2026-08-29

## Status

**DATABASE VALIDATION PASSED — слой работает в production; мерж кода в `main`
не выполнен**

2026-08-30: владелец выполнил все 7 шагов runbook на живой Supabase.
Итог: миграция применена; RLS включён без политик; EXECUTE на
`marketing_replace_day` только у `postgres`/`service_role`; обнаруженные
default-гранты Supabase у `anon`/`authenticated` отозваны (правка внесена и в
миграцию — `ae13124`); Security Advisor 0 errors и ни одного warning про
`marketing_*`; Edge Function `marketing-aggregates` задеплоена; backfill за 30
завершённых PT-дней записал 30 дней / 62 канальные / 491 постраничную строку;
повторный идентичный вызов не создал дублей и не изменил объём (идемпотентность
подтверждена на данных); cron-задача `marketing-aggregates` зарегистрирована
(`30 15 * * *`, active). Отчёт с дословными результатами:
`tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`.
Открытые пункты: повторная проверка грантов после `revoke` (`UNKNOWN`), первый
автоматический запуск по расписанию, и мерж ветки
`claude/marketing-analytics-aggregates` @ `ae13124` в `main` — мерж в этой
сессии заблокирован политикой окружения, ветка запушена и готова.

2026-08-30: Review 3 — исправление DST проверено по коду и тестам.
Решение: **CODE CORRECTION VERIFIED — READY FOR DATABASE VALIDATION**.
Отчёт: `tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`.
Это не `APPROVED_FOR_MAIN`: проверки migration/RLS/RPC/advisors на branch DB
ещё не выполнены.

2026-08-29 08:20 UTC: узкое замечание Review 2 (DST-unsafe target days)
исправлено — календарная арифметика от текущей PT-даты
(core.targetDaysBack); оба repro-кейса ревью дают корректный ответ;
11 новых тестов по полному списку ревью, 48/48 core-тестов PASS.
Branch: `claude/marketing-analytics-aggregates` @ `0b5ea3a`.
main/БД/функции не тронуты; дифф daily-stats пуст.
DB-execution проверки — pre-production gate (Supabase branch за владельцем).

2026-08-29: Review 2 — CHANGES_REQUESTED: одна точечная коррекция вычисления
списка Pacific calendar days при ручном/backfill запуске около DST.
Подробности: `tasks/reviews/2026-08-29-marketing-aggregates-review-2.md`.

2026-08-29 07:35 UTC: все 6 blocking findings Review 1 исправлены —
ответ: `tasks/reports/2026-08-29-marketing-aggregates-review1-response.md`.
Branch: `claude/marketing-analytics-aggregates` @ `64199f9`.
main/БД/функции по-прежнему не тронуты; дифф daily-stats пуст.
Оставшийся blocker (как разрешено ревью): проверки, требующие живой/branch
Supabase — за review/владельцем.

2026-08-29 07:10 UTC: Review 1 принят полностью — все 6 blocking findings
в работе (интерактивная Claude-сессия владельца).

Review 1: `tasks/reviews/2026-08-29-marketing-aggregates-review-1.md`.
Merge/apply/deploy запрещены до исправления blocking findings.

2026-08-29 06:25 UTC: discovery + реализация завершены — отчёт со всеми
доказательствами: `tasks/reports/2026-08-29-marketing-aggregates-discovery-handoff.md`.
Private branch: `claude/marketing-analytics-aggregates` @ `5e56eef`.
main НЕ изменён · БД НЕ изменена · функции НЕ деплоились.
Blocker для полной верификации: у сессии нет доступа к живой/branch Supabase —
SQL-применение, idempotency-на-БД, RLS/advisors проверяются на review.

2026-08-29 06:16 UTC: взято в работу интерактивной Claude-сессией владельца.
Фаза 1 (discovery) — в процессе; отчёт будет в `tasks/reports/`.
Ограничение, известное заранее: у сессии НЕТ доступа к живой Supabase
(ни ключей, ни branch database) — discovery по коду/SQL приватного репо;
состояние живой БД помечается UNKNOWN, тесты на живой БД — blocker для review.

## Objective

Создать в LICENA безопасный обезличенный аналитический слой ежедневных маркетинговых метрик поверх существующих данных Supabase.

Цель — дать ChatGPT возможность анализировать историческую динамику каналов, страниц и продуктовой воронки без доступа к email, UUID пользователей, device identifiers и индивидуальной истории пользователей.

Сначала провести аудит. Не применять миграцию и не деплоить Edge Function в production без review и статуса **APPROVED_FOR_MAIN**.

## Product context

В Supabase уже существуют:

- `public.page_views`;
- `public.social_stats`;
- `public.ga4_snapshots`;
- `public.gsc_snapshots`;
- `public.bing_snapshots`;
- `public.clarity_snapshots`;
- Edge Function `daily-stats`, формирующая ежедневное email-резюме;
- связанные auth, profiles, courses, trials и subscription records.

Текущий `daily-stats` содержит полезные агрегаты, но также собирает персональную секцию для владельца. Новые аналитические таблицы должны содержать только агрегаты и не должны копировать персональную секцию.

## Mandatory safety rules

1. Не ломать существующий ежедневный email и его текущих получателей.
2. Не менять auth, payments, Stripe, courses, RLS существующих таблиц или production cron без необходимости.
3. Не сохранять:
   - email;
   - user UUID;
   - device identifiers;
   - IP;
   - referrer с потенциальными sensitive query parameters;
   - индивидуальные paths конкретного пользователя;
   - персональную историю активности.
4. Не публиковать secrets, service-role keys, recipient addresses или PII в `licena-docs`, PR или logs.
5. Новые таблицы в exposed schema обязаны иметь RLS.
6. Не использовать permissive `TO authenticated` без ownership/authorization predicate.
7. Не создавать публичный `SECURITY DEFINER` API. Если privileged execution действительно необходимо, обосновать, держать функцию вне exposed schema, ограничить EXECUTE и проверить advisors.
8. Проценты/ratios не хранить, если они однозначно вычисляются из counts.
9. Не заявлять точную channel/payment attribution, если source linkage не подтверждён.
10. Все production schema/function/cron changes требуют **APPROVED_FOR_MAIN**.

## Phase 1 — discovery

До реализации опубликовать безопасный discovery report:

- какие фактические поля и события доступны;
- как `daily-stats` считает views, raw devices, engaged devices, signups, paid cohort и channel attribution;
- какие metrics можно воспроизвести точно;
- какие metrics являются приблизительными;
- какие события roadmap/application/course/checkout/payment реально существуют;
- какие timestamps пригодны для дневной агрегации;
- timezone rules;
- tester exclusion logic;
- attribution window;
- late-arriving payment behavior;
- существующие cron/schedules и idempotency constraints;
- data retention;
- потенциальное дублирование с GA4/GSC/Bing/Clarity snapshots;
- PII/security risks;
- минимальный implementation plan.

Если событие или связь не подтверждены кодом/схемой, пометить `UNKNOWN`; не создавать фиктивную метрику.

## Proposed schema

Имена могут быть скорректированы после discovery, но semantic contract должен сохраниться.

### 1. `marketing_daily_metrics`

Одна строка на календарный день в `America/Los_Angeles`.

Кандидатные поля:

- `day date primary key`;
- `page_views bigint`;
- `unique_devices_raw bigint`;
- `engaged_devices bigint`;
- `signed_in_users bigint`;
- `signups bigint`;
- `new_payers bigint` — только если дневной payment event подтверждён;
- `active_paid_subscriptions bigint` — snapshot;
- `active_trials bigint` — snapshot;
- `weekly_active_users bigint` — snapshot;
- `trial_to_paid_count bigint`;
- `computed_at timestamptz`;
- `source_version text` или эквивалент для аудита алгоритма.

Не хранить conversion percentages: вычислять при чтении через `nullif`.

### 2. `marketing_channel_daily`

Одна строка на день и нормализованный канал.

Предлагаемый ключ: `(day, channel)`.

Кандидатные поля:

- `day`;
- `channel`;
- `visitors`;
- `engaged_visitors`;
- `signups`;
- `paid_users` — только при подтверждённой attribution;
- `views`;
- `computed_at`.

Каналы должны использовать текущую проверенную taxonomy `daily-stats`, а не параллельную несовместимую классификацию.

### 3. `marketing_page_daily`

Одна строка на день и нормализованный path без sensitive query parameters.

Предлагаемый ключ: `(day, path)`.

Кандидатные поля:

- `day`;
- `path`;
- `views`;
- `engaged_visitors`;
- `signups` — только при подтверждённой attribution;
- `course_starts`;
- `checkout_starts`;
- `payments` — только при подтверждённой attribution;
- `computed_at`.

Не сохранять произвольные query strings. Разрешённые product identifiers должны нормализоваться отдельно после review.

## Product funnel events to audit

Проверить фактическое наличие и пригодность:

- roadmap started;
- roadmap intake completed;
- application assistant opened;
- exam-prep CTA clicked;
- course opened/started;
- trial started;
- checkout started;
- payment completed.

Если события отсутствуют, не добавлять instrumentation в этой задаче автоматически. Описать отдельный минимальный follow-up plan с privacy review.

## Aggregation behavior

- Timezone: `America/Los_Angeles`.
- Job должен быть идемпотентным: повторный запуск за день обновляет ту же строку/ключ.
- Поддержать безопасный bounded backfill для согласованного периода.
- Не сканировать неограниченную историю на каждом ежедневном запуске.
- Counts должны быть non-negative.
- Использовать `bigint` для растущих счётчиков.
- Индексы должны соответствовать реальным query patterns; не добавлять speculative indexes.
- Weekly/monthly ratios вычислять запросом из daily rows.
- Определить поведение при поздней оплате после trial: либо пересчитываем ограниченное окно, либо сохраняем cohort snapshot с явной семантикой.
- Tester exclusion должна совпадать с текущим подтверждённым product rule.

## Read model for ChatGPT

Предложить безопасный read-only интерфейс:

- предпочтительно private schema или security-invoker view;
- только aggregate columns;
- без direct access к source PII tables;
- documented example queries:
  - 7/30-day funnel;
  - channel comparison;
  - page performance;
  - week-over-week trend;
  - anomaly detection;
  - roadmap-to-course funnel, если события подтверждены.

Не предоставлять `anon` или всем `authenticated` широкий доступ. Точный access model должен быть описан в discovery.

## Implementation requirements

1. Отдельная private feature branch.
2. Migration file в принятом формате репозитория.
3. Idempotent DDL и documented rollback.
4. RLS включён до предоставления доступа.
5. Минимальные grants.
6. Aggregation logic должна переиспользовать или выделить текущую channel taxonomy без изменения поведения email.
7. Existing `daily-stats` продолжает отправлять прежний отчёт.
8. Если aggregation добавляется в `daily-stats`, ошибка записи aggregate не должна лишать владельца email; явно документировать fail-soft/fail-hard решение.
9. Предпочесть отдельный aggregation job, если это лучше изолирует email delivery.
10. Не применять migration/deploy до Owner approval.

## Verification

Обязательно предоставить:

- schema diff;
- SQL syntax/application test на branch/dev environment;
- idempotency test: два запуска дают одну строку на ключ;
- timezone boundary test;
- tester exclusion test;
- no-PII inspection;
- path sanitization test;
- channel taxonomy parity test;
- late-payment behavior test;
- RLS/grant test;
- Supabase database advisors after schema changes;
- sample aggregate queries;
- performance evidence for bounded aggregation;
- confirmation that existing daily email remains unchanged.

Если branch database недоступна, не тестировать на production; указать blocker.

## Required handoff in licena-docs

Публиковать только:

- status;
- discovery summary;
- private branch and commit SHA;
- changed areas;
- schema names/aggregate columns;
- tests/results;
- advisors results;
- migration name and **not applied/applied** status;
- Edge Function/cron changes and deployment status;
- preview/query examples containing synthetic or aggregate-only data;
- unresolved attribution questions;
- rollback procedure;
- explicit statements:
  - private main changed or unchanged;
  - production database changed or unchanged;
  - production functions/cron deployed or not deployed.

Не копировать private source, secrets, recipient addresses или PII.

## Acceptance criteria

- Discovery completed before implementation.
- No PII in new analytics tables or public report.
- Daily, channel and page aggregates have explicit stable semantics.
- Email report behavior and recipients are unchanged.
- Attribution limitations are documented.
- Job is idempotent and timezone-correct.
- RLS and grants verified.
- Query layer supports 7/30-day and channel/page analysis.
- Migration/Edge Function changes remain un-applied and un-deployed until approval.
- Claude publishes report and evidence in `licena-docs`.
- Final implementation status: **READY_FOR_REVIEW**.
- No merge/deploy to private main.

## Status flow

`READY_FOR_CLAUDE` → `IN_PROGRESS` → `READY_FOR_REVIEW`

Corrections: `CHANGES_REQUESTED` → `IN_PROGRESS` → `READY_FOR_REVIEW`.

Only Owner may authorize `APPROVED_FOR_MAIN`.

## Source References

- Owner approval in ChatGPT conversation, 2026-08-29.
- Current Supabase project schema and deployed `daily-stats` function — Claude must verify against private repository and current Supabase state.
- `tasks/WORKFLOW.md`.
- Supabase security/RLS and Postgres schema best practices.

## Verification Status

**Partially Verified** — current table/function existence was inspected read-only; event accuracy, attribution semantics and implementation paths require Claude discovery in the private repository.
