# Отчёт — валидация маркетинговых агрегатов на живой базе

Задача: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`
Runbook: `tasks/reports/2026-08-30-marketing-aggregates-runbook.md`
Гейт ревью: `tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`
Дата: 2026-08-30
Ветка кода: `claude/marketing-analytics-aggregates` @ `ae13124`

Все шаги на живой базе выполнены **владельцем** в Supabase Dashboard: у сессии
Claude нет доступа к Supabase (ни ключей, ни branch database). Ниже —
зафиксированные владельцем результаты каждого шага. Формулировки описывают
только то, что реально наблюдалось в выводе Dashboard.

## Шаг 1 — миграция

`supabase/sql/marketing-daily-aggregates.sql` выполнен в SQL Editor.
Созданы 4 таблицы (`marketing_daily_metrics`, `marketing_channel_daily`,
`marketing_page_daily`, `marketing_state_snapshots`) и функция
`marketing_replace_day(date, jsonb, jsonb, jsonb)`.

## Шаг 2 — RLS и гранты

Первичный прогон контрольных запросов runbook:

| Проверка | Результат |
|---|---|
| RLS на 4 таблицах | `true` на всех четырёх |
| Политики на этих таблицах | нет ни одной (deny-all для не-BYPASSRLS ролей) |
| EXECUTE на `marketing_replace_day` | только `postgres` и `service_role` |
| Права `anon` / `authenticated` на таблицах | **НЕ пусто** — обе роли имели SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER на всех 4 таблицах |

Диагноз последнего пункта: это не ошибка миграции, а default privileges
Supabase в схеме `public` — они выдаются автоматически на каждую **новую**
таблицу. Утечки данных не было: RLS без политик отказывает этим ролям в
любой строке. Но `TRUNCATE` **не подчиняется RLS**, поэтому право на него
у клиентских ролей — реальный риск потери данных.

Исправление: в миграцию добавлен явный `revoke all ... from anon,
authenticated` на все 4 таблицы (коммит `ae13124`); тот же SQL владелец
выполнил на живой базе — выполнено без ошибок.

Повторный прогон контрольного запроса прав после `revoke` в этой сессии
**не зафиксирован** — см. «Оставшаяся проверка» ниже.

## Шаг 3 — Database / Security Advisors

Security Advisor: **0 errors**, 22 warnings, 11 suggestions.
Полный список 22 warnings выгружен владельцем — **ни один не относится к
`marketing_*`**: все они касаются объектов, существовавших до этой задачи
(в том числе `rls_auto_enable`, Auth leaked-password protection, OTP expiry).
Новый слой не добавил ни одной ошибки и ни одного предупреждения.

## Шаг 4 — Edge Function

Функция `marketing-aggregates` задеплоена (Verify JWT включён).
Код собран в один файл из `core.ts` + `index.ts` без изменения логики.

## Шаг 5 — первичный backfill

Ручной вызов с `{"days": 30}` и service-role-ключом в заголовке. Ответ:

```
ok: true
days: 2026-07-30 … 2026-08-28 (30 дней)
days_written: 30
channel_rows: 62
page_rows: 491
snapshot_day: 2026-08-29
rows_read: 1970
source_version: v2
```

Подтверждает: авторизация по service-role проходит; окно чтения прочитано
целиком (fail-closed не сработал); текущий неполный PT-день в список не попал;
снепшот текущего состояния записан только за день запуска.

## Шаг 6 — идемпотентность и корректность

Функция вызвана **повторно с теми же параметрами**, затем выполнены
контрольные запросы:

| Проверка | Ожидание | Факт |
|---|---|---|
| строк в `marketing_channel_daily` / `marketing_page_daily` | без изменений | 62 / 491 |
| дублей ключа (day, channel) | 0 | 0 |
| дублей ключа (day, path) | 0 | 0 |
| строк за сегодняшний (неполный) день | 0 | 0 |
| строк / дней в `marketing_daily_metrics` | 30 / 30 | 30 / 30 |

Повторный прогон не создал дублей и не изменил объём данных — атомарная
замена дня через `marketing_replace_day` работает как задумано.

Канальная разбивка за окно (visitors / engaged / signups):

| channel | visitors | engaged | signups |
|---|---|---|---|
| direct | 578 | 60 | 6 |
| other | 152 | 69 | 11 |
| facebook | 2 | 0 | 0 |
| flyer | 1 | 1 | 0 |

## Шаг 7 — расписание

`supabase/sql/cron-marketing-aggregates.sql` выполнен (pg_cron + pg_net,
service-role-ключ в заголовке). Контрольный запрос:

```
jobname                | schedule    | active
marketing-aggregates   | 30 15 * * * | true
```

Запуск в 15:30 UTC — через 30 минут после ежедневного письма `daily-stats`
(15:00 UTC), чтобы сбой агрегатов не мог задеть отчёт владельца.

## Что осталось непроверенным

- **Повторная проверка грантов после `revoke`.** Команда выполнена без ошибок,
  но контрольный запрос прав (`information_schema.role_table_grants` по
  `anon`/`authenticated`) после неё в этой сессии не выполнялся. Статус —
  `UNKNOWN` до повторного прогона запроса «в» из runbook.
- **Первый автоматический запуск по расписанию.** На момент отчёта cron-задача
  зарегистрирована, но ещё не отрабатывала — проверяется после 15:30 UTC по
  `cron.job_run_details` и по появлению строки за очередной завершённый день.
- **Ротация ключа.** Service-role-ключ записан в теле cron-задачи в `cron.job`
  (тот же подход, что и `supabase/sql/cron-daily-stats.sql`). При ротации ключа
  скрипт нужно выполнить повторно, иначе задача начнёт получать 403.

## Дополнение 2026-08-30 (позже в тот же день)

- **Санитизация путей проверена на живых данных:** `select count(*) from
  marketing_page_daily where path like '%?%'` → **0**. Query-строк в хранимых
  путях нет. Пункт закрыт.
- **Cron содержал нерабочий ключ.** При первой настройке в тело задачи попал
  ключ вместе с угловыми скобками плейсхолдера (`<...>`), из-за чего шлюз
  Supabase отвечал `401 UNAUTHORIZED_INVALID_JWT_FORMAT`. Обнаружено при
  попытке повторного бэкфилла; задача пересоздана с корректным секретным
  ключом, проверено запросом (`command like '%<%'` → false,
  `command like '%Bearer sb_secret_%'` → true). До исправления ночной запуск
  15:30 UTC гарантированно падал бы — за весь период он ни разу не отработал
  успешно.
- **Бэкфилл после разделения каналов** выполнен повторно: ответ `200`,
  `"ok": true`, 30 дней начиная с 2026-07-30. Итоговая разбивка по каналам —
  `15_METRICS.md`, аддендум 2026-08-30.

Остаются незакрытыми: повторная проверка грантов после `revoke`, негативные
ветки авторизации (без ключа / anon / пользовательский JWT), проверка отката
(решение владельца 2026-08-30 — пропустить, риск зафиксирован), первый
успешный автоматический запуск по расписанию.

## Source References

- `LLICENA:supabase/sql/marketing-daily-aggregates.sql` (ветка `claude/marketing-analytics-aggregates` @ `ae13124`)
- `LLICENA:supabase/sql/cron-marketing-aggregates.sql`
- `LLICENA:supabase/functions/marketing-aggregates/core.ts`
- `LLICENA:supabase/functions/marketing-aggregates/index.ts`
- `LLICENA:scripts/test-marketing-core.mjs`, `LLICENA:scripts/check-channel-parity.js`
- `tasks/reports/2026-08-30-marketing-aggregates-runbook.md`
- `tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`
- Результаты выполнения владельцем в Supabase Dashboard, 2026-08-30 (выгрузки
  запросов и ответы функции, приведённые выше дословно)

## Verification Status

**Partially Verified** — шаги 1, 3, 4, 5, 6, 7 подтверждены выводом Supabase
Dashboard, приведённым в отчёте дословно. Шаг 2 подтверждён частично: RLS,
отсутствие политик и EXECUTE проверены запросом; состояние грантов
`anon`/`authenticated` **после** `revoke` помечено `UNKNOWN` — контрольный
запрос после команды не выполнялся. Все проверки на живой базе выполнял
владелец: у сессии Claude нет доступа к Supabase.
