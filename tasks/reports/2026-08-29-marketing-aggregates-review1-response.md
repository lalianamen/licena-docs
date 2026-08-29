# REPORT — Marketing Aggregates: ответ на Review 1 (все 6 findings исправлены)

Задача: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`
Review: `tasks/reviews/2026-08-29-marketing-aggregates-review-1.md` · Дата: 2026-08-29
Private branch: `claude/marketing-analytics-aggregates` @ `64199f9` (v1 был `5e56eef`)

## Явные заявления

- **Private `main` — UNCHANGED.** · **Production database — UNCHANGED.** · **Functions/cron — NOT DEPLOYED.**
- Дифф `daily-stats` — по-прежнему **пуст** (проверено `git diff main -- supabase/functions/daily-stats/` = 0 строк).

## Исправления по каждому blocking finding

### F1 — Historical snapshot corruption → исправлено
`active_paid_subscriptions` / `active_trials` / `trial_to_paid_count` **убраны из per-day строк** — по схеме они не реконструируемы as-of прошлого дня (user_courses мутируется grant'ами). Новая таблица `marketing_state_snapshots` (day PK): одна наблюдённая запись за день запуска, **backfill запрещён семантикой** (комментарий в DDL). `weekly_active_users` остался в daily-строке, но теперь **as-of конца каждого target-дня** (окно 7×24ч до конца дня — реконструируемо из page_views).

### F2 — Non-deterministic attribution → исправлено
Правило зафиксировано: канал устройства для дня D = первый канальный просмотр в **фиксированные 28 дней до конца D** (`ATTRIBUTION_LOOKBACK_DAYS`, не зависит от параметра `days`). Окно чтения в index.ts всегда покрывает lookback самого старого target-дня; лишние строки игнорируются ядром. Тесты: тот же день на узком vs широком наборе строк и при перемешанном порядке → байт-идентичные daily/channel/page результаты (3 теста PASS).

### F3 — Privileged function authorization → исправлено
Принятый в проекте паттерн gsc-sync повторён дословно: bearer/apikey должен совпасть с service-role ключом, иначе capability-probe через GoTrue admin endpoint (200 только для service-уровня); иначе **403**, никакие credentials не логируются и не возвращаются. Negative-тесты с реальными anon/user токенами — часть DB-blocker'а (нужен живой проект), логика гейта — прямая копия уже принятого кода.

### F4 — Silent 50k truncation → исправлено
Пагинация `range()` по 1000 строк со стабильным порядком (`at, id`), бюджет 400k. При исчерпании — **fail closed**: HTTP 507, `{reason:"source_window_truncated", rows_read, max_rows}` (без PII), **ничего не записывается**. 5 тестов пагинации PASS (все страницы, граница cap,超 cap, ошибка, короткая страница).

### F5 — Stale rows → исправлено
Новая функция БД `marketing_replace_day(day, daily, channels, pages)` — security **invoker**, `set search_path=public`, delete+insert трёх таблиц **в одной транзакции на день**; EXECUTE отозван у public/anon/authenticated, выдан service_role. Ядро возвращает ПОЛНЫЙ набор строк дня (тест: исчезнувший источник → ключ исчезает из результата, набор полон). Partial failure бэкфилла оставляет предыдущие дни целиком записанными, текущий — целиком старым.

### F6 — DB constraints → исправлено
`CHECK (col >= 0)` на каждом счётчике всех 4 таблиц; `channel in ('direct','flyer','facebook','instagram','telegram','tiktok','other')` (закрытый набор = текущая taxonomy); `char_length(path) between 1 and 200`.

## Тестируемость: логика вынесена в чистое ядро
`supabase/functions/marketing-aggregates/core.ts` (pure, без Deno) импортируется тестами напрямую (`node --experimental-strip-types`) — тестируется **production-код, не копия**. `index.ts` — только I/O-обёртка (auth gate, пагинация, RPC-записи).

## Verification (выполнено здесь)

| Проверка | Результат |
|---|---|
| `scripts/test-marketing-core.mjs` — 37 проверок: determinism F2 (×4), snapshot-семантика F1 (×4), replace-полнота F5 (×2), fail-closed пагинация F4 (×5), DST-границы PT (2026-03-08 spring-forward, 2025-11-02 fall-back, лето/зима, до/после границы — ×6), классификатор (×3), tester exclusion, non-negative F6 | **ALL PASS** |
| Channel taxonomy parity (daily-stats ↔ core.ts, побайтно) | PASS |
| TS parse (node strip-types) index.ts + core.ts | PASS |
| `node scripts/verify.js` | 0 ошибок, 138 JS OK |
| Дифф daily-stats | пуст |

## Оставшийся blocker (чётко ограничен, как разрешено ревью)
Требуют живой/branch Supabase (у сессии нет доступа; на production не тестируем): применение SQL + advisors, два идентичных запуска на БД, RLS/grants-тест ролями, negative-auth с настоящими project-токенами, cap-тест на реальном объёме. Изолированная branch DB — за владельцем (стоимость).

## Статус
**READY_FOR_REVIEW** (Review 2). Merge/apply/deploy — только после APPROVED_FOR_MAIN.
