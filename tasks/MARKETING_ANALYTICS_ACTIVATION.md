# TASK — Activate LICENA Marketing Analytics Data Layer

## Status

**OBSOLETE ON ARRIVAL — задача поставлена под состояние, которого уже нет**

2026-08-30 (запись Claude, append-only): активация, которую эта задача просит
ПОДГОТОВИТЬ, к моменту её постановки уже ВЫПОЛНЕНА владельцем в production, а
код смержен в `main` по его прямому указанию. Поэтому Phase 1 (валидация на
изолированной среде) и Phase 2 (подготовка пакета активации без исполнения)
неприменимы в исходной формулировке.

Устаревшие предпосылки: reviewed commit `0b5ea3a` → актуальный `ae13124`;
«production does not contain the marketing aggregate tables» → содержит все 4
таблицы, функция задеплоена, cron `30 15 * * *` активен, выполнен backfill за
30 завершённых PT-дней; правила «не мержить в main» и «не применять без
`APPROVED_FOR_MAIN`» перекрыты прямым распоряжением владельца, отданным ДО
постановки этой задачи.

Полный разбор по всем 15 пунктам Phase 1 с доказательствами, фактическое
состояние production и ответ на раздел Data quality follow-up:
`tasks/reports/2026-08-30-marketing-analytics-activation-response.md`.

Осталось незакрытым (требует действий владельца — у AI-сессии нет доступа к
Supabase): негативные ветки авторизации (пункт 7), повторная проверка грантов
после `revoke` (пункт 4), проверка отката на изолированной branch database
(пункт 14), контрольный запрос санитизации `path` на живых данных (пункт 10),
первый автоматический запуск cron. Решение по разделению бакета `other` на
`search`/AI — за владельцем: правка затрагивает и ежедневное письмо.

## Context

The implementation already exists in the private repository:

- repository: `lalianamen/LLICENA`;
- branch: `claude/marketing-analytics-aggregates`;
- reviewed commit: `0b5ea3a`;
- code review: `tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`.

The production Supabase project currently does **not** contain the proposed
marketing aggregate tables. ChatGPT can calculate temporary read-only metrics
from raw tables, but there is no persistent daily analytical history yet.

Existing raw sources include `page_views`, `profiles`, `course_trials`,
`user_courses`, and GA4/GSC/Bing/Clarity snapshots. `social_stats` currently
has no rows.

## Objective

Prepare the marketing analytics implementation for a safe owner-approved
production activation so ChatGPT can analyze LICENA through anonymized daily
aggregates instead of querying PII-bearing source tables.

Do not redesign the implementation. Continue from commit `0b5ea3a` and make
only changes required by validation findings.

## Mandatory safety rules

1. Do not merge to private `main`.
2. Do not apply migrations, deploy Edge Functions, create/change cron, or write
   backfill rows in production without explicit `APPROVED_FOR_MAIN`.
3. Do not test destructive or write behavior against production.
4. Do not expose service-role keys, JWTs, recipient addresses, email, UUIDs,
   device identifiers, IP addresses, raw referrers, PII, or private code in
   `licena-docs`, logs, screenshots, issues, or reports.
5. Do not modify existing `daily-stats` behavior or recipients.
6. Preserve auth, payments, courses, analytics, RLS, and production routes.
7. All analytical output and fixtures must be synthetic or aggregate-only.

## Phase 1 — validate without production changes

Use an isolated Supabase branch database, local Supabase stack, or disposable
development database. If none is available, stop and report the exact blocker;
do not substitute production.

Validate:

1. Apply the proposed SQL from a clean compatible schema.
2. Confirm these tables and their exact stable semantics:
   - `marketing_daily_metrics`;
   - `marketing_channel_daily`;
   - `marketing_page_daily`;
   - `marketing_state_snapshots`.
3. Verify all public/exposed tables have RLS enabled.
4. Verify `anon` and normal `authenticated` users cannot read or write the
   aggregate tables.
5. Verify only the intended privileged execution path can replace a day.
6. Verify the atomic replace RPC:
   - two runs for the same date leave one row per key;
   - removed source channel/page keys do not survive recomputation;
   - failure cannot leave a partially replaced day.
7. Verify service-role authorization:
   - missing credentials → denied;
   - anon key → denied;
   - user JWT → denied;
   - intended service credential → accepted.
8. Verify bounded pagination and fail-closed behavior.
9. Verify DST/calendar behavior and rerun the full core test suite.
10. Verify path sanitization and absence of query strings/PII.
11. Verify tester exclusion.
12. Verify channel taxonomy parity with `daily-stats`.
13. Run database security/performance advisors and address relevant findings.
14. Verify rollback on the isolated environment.
15. Confirm `daily-stats` has no diff and still behaves independently.

## Phase 2 — prepare activation package, do not execute

Prepare an exact owner-reviewable activation plan containing:

1. migration order;
2. Edge Function deployment command/settings;
3. JWT/service-role configuration requirements without secret values;
4. proposed cron schedule in `America/Los_Angeles` semantics;
5. bounded initial backfill period and estimated source-row volume;
6. verification queries using aggregate-only output;
7. rollback steps for migration, function, and cron;
8. monitoring/failure signals;
9. confirmation that reruns are idempotent;
10. expected Supabase cost/branch requirement, if any.

Do not execute this phase until the owner changes the task status to
`APPROVED_FOR_MAIN`.

## Data quality follow-up

Audit why most recent traffic is classified as `direct`, `other`, or
unattributed and why `social_stats` is empty. Do not change tracking in this
task. Provide a minimal follow-up proposal for consistent per-channel links,
using the current confirmed `src` taxonomy and/or a backward-compatible UTM
mapping. Do not claim exact attribution where linkage is not confirmed.

## Required report in licena-docs

Publish a concise safe report under `tasks/reports/` with:

- status;
- private branch and commit SHA;
- environment used for validation;
- migration applied/not applied and where;
- test matrix with pass/fail evidence;
- RLS/grant results;
- advisor results;
- idempotency/atomicity results;
- authorization results;
- DST and no-PII results;
- proposed backfill window;
- activation and rollback summary;
- unresolved blockers;
- explicit production state:
  - private main changed/unchanged;
  - production database changed/unchanged;
  - production functions/cron deployed/not deployed.

Do not publish private source code, secrets, PII, raw user-level data, or
sensitive implementation details.

## Exit status

If isolated validation succeeds: **READY_FOR_REVIEW**.

If no safe isolated environment is available: **CHANGES_REQUESTED** with the
blocker and required owner action.

Never set `APPROVED_FOR_MAIN`; only the owner may do that.
