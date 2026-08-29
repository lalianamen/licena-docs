# REVIEW 2 — Marketing Analytics Daily Aggregates

Дата: 2026-08-29  
Reviewed branch: `claude/marketing-analytics-aggregates` @ `64199f9`

## Decision

**CHANGES_REQUESTED — one narrow correction**

Review 1 findings F1–F6 are accepted as fixed in code. Do not merge/apply/deploy yet.

## Remaining blocking finding: target-day generation is not DST-safe

The aggregation core correctly calculates Pacific day boundaries, but the I/O wrapper builds `targetDays` by subtracting fixed `864e5` intervals from `Date.now()` and then converting each timestamp to `America/Los_Angeles`.

A Pacific calendar day is not always 24 hours. Around DST transitions and during manual calls near the local midnight boundary, this can skip the most recent completed Pacific day or include the current incomplete day.

Reproduced examples using the production formula:

- `2026-03-09T07:30:00Z` (00:30 PDT after spring-forward): a 3-day request produces Mar 5–7 instead of completed Mar 6–8.
- `2025-11-03T07:30:00Z` (23:30 PST on Nov 2): it includes Nov 2 although that Pacific day is not yet complete.

The scheduled 15:30 UTC cron is not affected in these examples, but manual/backfill calls are part of the supported contract and must be time-of-call independent.

## Required correction

1. Derive the current Pacific calendar date first.
2. Generate preceding calendar dates using calendar arithmetic, not elapsed 24-hour durations.
3. Keep `ptDayEndUtcMs` (or equivalent timezone-aware boundary logic) for source windows.
4. Add tests for target-day generation:
   - immediately before and after spring-forward local midnight;
   - immediately before and after fall-back local midnight;
   - scheduled cron time;
   - arbitrary manual call time;
   - no duplicate dates;
   - never include the current incomplete Pacific day;
   - exactly N consecutive completed Pacific dates.

## Verification note

No Supabase branches currently exist for project `licena`. Database application, advisors, RLS-role tests and real-token negative-auth tests therefore remain a pre-production gate. Do not create a paid branch or test on production without Owner approval.

## Status

`READY_FOR_REVIEW (Review 2)` → **`CHANGES_REQUESTED`**

After the narrow correction, publish commit and test output, then return to `READY_FOR_REVIEW (Review 3)`.
