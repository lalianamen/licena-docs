# RUNBOOK — применение Marketing Analytics Daily Aggregates

Задача: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md` · Review 3: код проверен
(`tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`)
Ветка: `claude/marketing-analytics-aggregates` @ `94c9157` · Дата: 2026-08-30

Выполняет **владелец** (у AI-сессии нет доступа к Supabase). Порядок важен.
После шага 6 пришлите вывод проверочных запросов — сессия смержит ветку в
`main`, поставит `DONE` и обновит документацию.

## Шаг 1 — таблицы и функция замены дня

Supabase → SQL Editor → вставить целиком файл
`supabase/sql/marketing-daily-aggregates.sql` (из ветки; копия приложена в чат)
→ Run. Скрипт идемпотентен, существующие таблицы не трогает.

Создаётся: `marketing_daily_metrics`, `marketing_channel_daily`,
`marketing_page_daily`, `marketing_state_snapshots`, функция
`marketing_replace_day()`.

## Шаг 2 — проверка схемы, RLS и грантов

```sql
-- (а) 4 таблицы, у всех RLS = true
select tablename, rowsecurity from pg_tables
where schemaname='public' and tablename like 'marketing_%' order by 1;

-- (б) политик НЕТ ни одной (доступ только service-role) — ожидается 0 строк
select tablename, policyname from pg_policies
where schemaname='public' and tablename like 'marketing_%';

-- (в) у anon/authenticated НЕТ прав на таблицы — ожидается 0 строк
select grantee, table_name, privilege_type from information_schema.role_table_grants
where table_schema='public' and table_name like 'marketing_%'
  and grantee in ('anon','authenticated');

-- (г) EXECUTE на функции — только service_role
select grantee, privilege_type from information_schema.routine_privileges
where routine_name='marketing_replace_day';
```

## Шаг 3 — Database Advisors

Dashboard → Advisors → Security и Performance.
**Ожидаемое предупреждение:** «RLS enabled but no policies» на четырёх новых
таблицах — это НАМЕРЕННО (тот же приём, что у существующей `social_stats`:
доступ только у service-role). Любые ДРУГИЕ новые предупреждения — присылайте.

## Шаг 4 — Edge Function

Dashboard → Edge Functions → Create a new function → имя ровно
`marketing-aggregates` → вставить код из приложенного файла
`marketing-aggregates-single-file.ts` → Deploy. Verify JWT оставить ВКЛ.
Новых секретов не нужно (`SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY`
подставляются автоматически).

Функция отвечает **403** на любой ключ, кроме service-role — так и задумано.

## Шаг 5 — первичный backfill (30 дней)

SQL Editor, подставив свои значения (ключ никуда не сохраняется):

```sql
select net.http_post(
  url     := 'https://<PROJECT_REF>.supabase.co/functions/v1/marketing-aggregates',
  headers := jsonb_build_object('Content-Type','application/json',
                                'Authorization','Bearer <SERVICE_ROLE_KEY>'),
  body    := '{"days": 30}'::jsonb);
```

Подождать ~10–30 секунд. Если функция вернёт `{"ok":false,...}` —
прислать текст: она сознательно **не пишет ничего**, когда не может прочитать
окно данных целиком.

## Шаг 6 — проверки данных и идемпотентности

```sql
-- (а) данные появились
select day, page_views, unique_devices_raw, engaged_devices, signed_in_users,
       signups, weekly_active_users, source_version
from marketing_daily_metrics order by day desc limit 10;

-- (б) снимок состояния за сегодня
select * from marketing_state_snapshots order by day desc limit 3;

-- (в) каналы и страницы
select channel, sum(visitors) v, sum(signups) s from marketing_channel_daily group by 1 order by v desc;
select path, sum(views) v from marketing_page_daily group by 1 order by v desc limit 10;
```

Затем **повторить вызов из шага 5 ещё раз** и проверить идемпотентность:

```sql
-- (г) один день = одна строка; ожидается равенство
select count(*) as rows, count(distinct day) as days from marketing_daily_metrics;

-- (д) дублей ключей нет — ожидается 0 строк
select day, channel, count(*) from marketing_channel_daily group by 1,2 having count(*)>1;
select day, path, count(*) from marketing_page_daily group by 1,2 having count(*)>1;

-- (е) сегодняшний (неполный) день НЕ попал — ожидается 0
select count(*) from marketing_daily_metrics
where day >= (now() at time zone 'America/Los_Angeles')::date;
```

## Шаг 7 — расписание

SQL Editor → файл `supabase/sql/cron-marketing-aggregates.sql` (подставить
PROJECT_REF и SERVICE_ROLE_KEY) → Run. Запуск в 15:30 UTC — через 30 минут
после ежедневного письма, чтобы никогда на него не влиять.

```sql
select jobname, schedule, active from cron.job where jobname='marketing-aggregates';
```

## Что прислать сессии

Вывод пунктов 2(а–г), 6(а,г,д,е) и ответ функции из шага 5. Этого достаточно,
чтобы закрыть database-validation gate из Review 3.

## Откат

Всё обратимо: `drop` четырёх таблиц и функции (готовые команды — в конце
`supabase/sql/marketing-daily-aggregates.sql`), `cron.unschedule('marketing-aggregates')`,
удаление Edge Function в Dashboard. Исходные данные (`page_views`,
`user_courses`, `course_trials`, `auth.users`) не изменяются ни на одном шаге —
слой только читает их.
