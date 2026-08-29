# REVIEW — Marketing Analytics Daily Aggregates

Дата: 2026-08-29  
Reviewer: ChatGPT  
Task: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`  
Reviewed private branch: `claude/marketing-analytics-aggregates`  
Reviewed commit: `5e56eef` (one commit ahead of private `main@037bdd9`)

## Decision

**CHANGES_REQUESTED**

Не merge. Не применять SQL. Не deploy Edge Function/cron.

Discovery и безопасность данных в целом подготовлены хорошо: private main и production не изменены; email-функция не затронута; новые таблицы не содержат PII; неподтверждённые payment/page funnel metrics не выдуманы. Однако текущая реализация может сохранять исторически неверные или неполные агрегаты и недостаточно защищает privileged Edge Function.

## Blocking findings

### 1. Historical snapshot corruption

`active_paid_subscriptions`, `active_trials`, `weekly_active_users` и `trial_to_paid_count` вычисляются один раз в момент запуска и одинаковыми текущими значениями записываются во все пересчитываемые дни.

При стандартном повторном пересчёте последних трёх дней предыдущие дневные snapshots будут перезаписываться сегодняшним состоянием. При 30-дневном backfill все 30 дней получат одинаковый текущий snapshot. Это не историческая динамика.

Required correction:

- либо вычислять каждую snapshot-метрику as-of end-of-each-target-day из подтверждённых timestamps;
- либо не записывать её в исторические day rows и вынести current-state snapshots в отдельную таблицу/датированную запись с честной семантикой;
- метрики, для которых прошлое состояние невозможно восстановить, не backfill-ить и явно пометить ограничение.

### 2. Non-deterministic channel attribution

Окно определения первого канала зависит от входного `days`: фактически `days + 7`. Поэтому один и тот же target day может получить другой канал при запуске с `days=3` и `days=30`.

Это нарушает стабильную семантику и идемпотентность результата относительно параметра backfill.

Required correction:

- определить фиксированное attribution rule для каждого target day;
- результат одного target day должен быть одинаков при одиночном пересчёте и составе более широкого backfill;
- добавить тест: same target day, `days=1` vs multi-day backfill → identical channel rows.

### 3. Privileged function authorization

`verify_jwt=true` само по себе не является достаточным authorization gate для функции, использующей service role: project anon/authenticated JWT также может пройти gateway JWT verification. В проекте это уже было отдельно зафиксировано и исправлялось для других sync-функций.

Required correction:

- добавить явную проверку вызывающего cron/оператора по принятому в проекте защищённому паттерну;
- не полагаться только на `verify_jwt`;
- не логировать и не возвращать credentials;
- добавить negative authorization tests.

### 4. Silent 50k truncation

Source query ограничена 50 000 rows, после чего функция продолжает расчёт и записывает заниженные counts без признака incomplete/truncated.

Required correction:

- paginate bounded source reads; либо
- fail closed, если cap достигнут, и не записывать aggregates;
- response/log должен содержать безопасный diagnostic без PII;
- добавить cap-boundary test.

### 5. Stale channel/page rows

Upsert обновляет присутствующие keys, но не удаляет rows для target day, которые исчезли после изменения source data/taxonomy/algorithm. Повторный расчёт может оставить устаревшие channel/path rows.

Required correction:

- обеспечить atomic replace per target day или другой доказуемый способ полного reconciliation;
- partial failure не должен оставлять смешанное состояние старой и новой версии;
- добавить тест, где key исчезает после recompute.

### 6. Missing database-level count constraints

Задача требует non-negative counts, но schema не содержит `CHECK (... >= 0)`.

Required correction:

- добавить database CHECK constraints для всех count columns;
- добавить допустимые channel constraints или обоснованный extensible validation mechanism;
- проверить oversized path/channel handling.

## Required verification before next review

1. SQL application in isolated Supabase branch/dev database, not production.
2. Two identical executions → identical rows and no duplicates.
3. Same target day in narrow and wide backfill → identical daily/channel/page result.
4. Historical snapshot semantics test.
5. Atomic stale-row removal/reconciliation test.
6. Pacific timezone boundary tests covering DST-relevant dates.
7. Negative authorization tests for missing, anon and ordinary authenticated tokens.
8. 50k-cap/pagination test.
9. RLS/grants test for anon, authenticated and service role.
10. Database advisors after applying schema to isolated environment.
11. No-PII inspection.
12. Existing `daily-stats` diff remains empty.

If an isolated Supabase branch cannot be provisioned without owner cost approval, Claude should implement the deterministic corrections and report the remaining DB execution checks as a clearly scoped blocker. Do not test by applying to production.

## Non-blocking notes

- Separate Edge Function is the correct isolation choice for preserving daily email delivery.
- Omitting unprovable payment attribution is correct.
- Primary-key order is suitable for stated date-window queries.
- A dedicated read-only analyst role is optional; current owner-authorized Supabase connection can query aggregates after approval without exposing them to anon/authenticated users.

## Status transition

`READY_FOR_REVIEW` → **`CHANGES_REQUESTED`**

After corrections:

`CHANGES_REQUESTED` → `IN_PROGRESS` → `READY_FOR_REVIEW`

Only Owner may set `APPROVED_FOR_MAIN`.

## Verification Status

**Code-reviewed, database-unverified.** Three branch artifacts and the one-commit diff were inspected. No production mutation was performed.
