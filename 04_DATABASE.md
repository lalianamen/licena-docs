# 04 — База данных (Supabase Postgres)

Последняя сверка: 2026-08-05 (полная) · 2026-08-28 (точечная: License Roadmap) · 2026-08-29 (точечная: Application Assistant + Adaptive Intake v2)

Дополнение 2026-08-28 (`0d7fcfb` осн. репо, `supabase/sql/license-roadmaps.sql`):
новые таблицы фичи License Roadmap (Phase 1) — SQL идемпотентен, применяется
владельцем в SQL Editor; до применения фича работает на localStorage.
- `public.license_roadmaps` — id uuid PK (gen_random_uuid), user_id → auth.users
  (cascade), state text, license_type text (id из `js/paths.js` LICENSE_PATHS),
  license_not_sure boolean, answers jsonb (ответы анкеты), status
  ('active'/'archived'/'issued'), current_step text, progress_percent int,
  created_at/updated_at; индекс (user_id, updated_at desc). Один пользователь
  может иметь несколько роадмапов (штат × лицензия).
- `public.license_roadmap_steps` — id uuid PK, roadmap_id → license_roadmaps
  (cascade), step_key text, step_order int, status text (6 значений:
  not_started/action_needed/in_progress/waiting/review_needed/completed),
  completed_at, metadata jsonb, created_at/updated_at; unique (roadmap_id,
  step_key).
- RLS: обе таблицы owner-only на все операции (auth.uid() = user_id;
  для steps — через exists по родительскому роадмапу).
- Триггеры `touch_updated_at` (общая функция public.touch_updated_at)
  на update обеих таблиц.
- Контент шагов/правила НЕ в БД — в клиентском конфиге
  `js/roadmap/roadmap-config.js` (добавление штата не меняет схему).
Статус применения в Supabase на 2026-08-28: UNKNOWN — за владельцем.

Дополнение 2026-08-29 (`2c66f13` осн. репо, `supabase/sql/license-applications.sql`):
таблица фичи Application Assistant (Phase 1) — SQL идемпотентен, применяется
владельцем в SQL Editor; до применения фича работает на localStorage.
- `public.license_applications` — id uuid PK (gen_random_uuid), user_id →
  auth.users (cascade), state text ('ca'), application_type text ('original'),
  entity_type text, classification_id text, status text (check:
  draft/review_needed/ready_for_submission/submitted_user_confirmed/
  under_review/correction_requested/accepted), completion_percent int (0–100),
  sections jsonb (подготовленные ответы по разделам), submitted_user_at date
  (ДАТА СО СЛОВ ПОЛЬЗОВАТЕЛЯ — не подтверждение CSLB), correction jsonb,
  created_at/updated_at; индекс (user_id, updated_at desc).
- RLS: owner-only на select/insert/update/delete (auth.uid() = user_id).
- Триггер `license_applications_touch` (общая функция touch_updated_at).
- ПРИВАТНОСТЬ: sections НИКОГДА не содержит SSN/ITIN, дату рождения, номер
  водительского удостоверения или тексты disclosure-объяснений — клиент
  (`js/roadmap/app-application.js`) эти поля сознательно не собирает; они
  заполняются пользователем только в официальной форме CSLB.
Статус применения в Supabase на 2026-08-29: UNKNOWN — за владельцем.

Дополнение 2026-08-29 (`037bdd9` осн. репо, `supabase/sql/license-roadmaps-v2.sql`):
Adaptive Intake v2 — 12 новых колонок в `public.license_roadmaps`
(идемпотентный `ADD COLUMN IF NOT EXISTS`; RLS и политики НЕ меняются):
- Даты (тип date): `application_submitted_at`, `correction_received_at`,
  `fingerprint_scheduled_at`, `fingerprint_completed_at`, `law_exam_date`,
  `trade_exam_date`, `license_issued_at` — все со слов пользователя.
- `entity_type` text ('sole_owner'/'partnership'/'corporation'/'llc'/'tribal'),
  `entity_name` text, `dba` boolean default false, `license_number` text,
  `reminder_opt_in` boolean default false (напоминания об экзаменах —
  ОТДЕЛЬНО от маркетинговых согласий).
- Обратная совместимость: клиент (`js/roadmap/app-roadmap.js` dbSave)
  сначала пишет расширенную строку, при ошибке (колонок ещё нет) — строку
  v1; полные ответы в любом случае остаются в `answers` jsonb. Откат —
  закомментированный `DROP COLUMN IF EXISTS` в том же файле (обратимая
  миграция; данные дублируются в jsonb).
Статус применения в Supabase на 2026-08-29: UNKNOWN — за владельцем.
Источник: SQL-файлы репозитория `lalianamen/llicena@main` (все прочитаны
полностью). ВАЖНО: файлы — это скрипты, которые владелец запускает вручную в
Supabase SQL Editor; фактическое текущее состояние живой БД по репозиторию не
проверяемо (см. «Verification Status»).

## Таблицы

### `public.profiles` (`docs/schema.sql`)
Расширение `auth.users`; создаётся при регистрации.

| Колонка | Тип | Примечание |
|---|---|---|
| `id` | uuid PK → `auth.users(id)` on delete cascade | |
| `name` | text | |
| `target_exam` | text default `'cslb-law'` | |
| `lang` | text default `'en'` | язык интерфейса |
| `created_at` | timestamptz default now() | |
| `is_tester` | boolean not null default false | флаг тестера (`supabase/sql/tester-account.sql`, продублировано в `devices_anti_sharing.sql`) |
| `stripe_customer_id` | text | один Stripe-customer на аккаунт (`supabase/sql/stripe-payments.sql` A1) |

RLS: `own select` / `own insert` / `own update` по `auth.uid() = id`
(`docs/schema.sql`). Примечание из `tester-account.sql`: политика own-update
позволяет пользователю самому выставить `is_tester` — это раскрывает только
карточку тест-курса, не платные вопросы («flag is a curtain, not a lock»).

### `public.devices` (`docs/schema.sql` + `supabase/devices_anti_sharing.sql`)

| Колонка | Тип |
|---|---|
| `id` | uuid PK default gen_random_uuid() |
| `user_id` | uuid → auth.users, not null |
| `device_token` | text not null; unique(user_id, device_token) |
| `added_at` | timestamptz default now() |
| `last_seen_at` | timestamptz default now() (добавлена anti-sharing-скриптом) |

RLS: `own select` остаётся; политики insert/delete **удалены** — записи только
через `register_device()` (security definer), чтобы лимит нельзя было обойти из
браузера (`devices_anti_sharing.sql` §2).

### `public.user_courses` (`docs/schema.sql` + `supabase/sql/subscriptions-schema.sql`)
Права доступа к курсам (entitlements).

| Колонка | Тип | Примечание |
|---|---|---|
| `id` | uuid PK | |
| `user_id` | uuid → auth.users, not null | unique(user_id, course_id) |
| `course_id` | text not null | ключ курса из каталога |
| `status` | text not null default `'active'` | `'active' \| 'inactive'` (миграция `docs/migration-2026-06-22-course-status.sql`) |
| `activated_at` | timestamptz default now() | |
| `started_at` | timestamptz default now() | subscriptions-schema |
| `expires_at` | timestamptz | null = без срока (free tier) |
| `auto_renew` | boolean not null default false | зеркалит `!cancel_at_period_end` |
| `stripe_subscription_id` | text | связь со Stripe-подпиской |

RLS после платёжного флипа (`stripe-payments.sql` SECTION B): все старые
INSERT/UPDATE-политики удаляются циклом по `pg_policies`; создаются
`courses: own insert (free tier only)` и `courses: own update (free tier only)` —
самозапись разрешена **только для не-платных** course_id (список из 14 платных
захардкожен в политике). SELECT и DELETE остаются `own`. Платные строки пишет
только `stripe-webhook` (service role, мимо RLS) и `start_trial()`.

История бета-строк: `expires_at = '2026-08-01T07:00:00Z'` (Aug 1, 00:00 PT)
проставлен бета-строкам платных банков — сначала 10 банкам
(`subscriptions-schema.sql`), затем `c27-landscaping` (`stripe-payments.sql` A2)
и остальным добавленным позже (SECTION B2, кроме строк со
`stripe_subscription_id`).

### `public.bank_questions` (`supabase/sql/bank-questions-schema.sql`)
Платные банки вопросов (перенесены из публичной статики).

| Колонка | Тип | Примечание |
|---|---|---|
| `course_id` | text not null | PK (course_id, lang, qid) |
| `lang` | text not null default `'en'` | `'en' \| 'es' \| 'ru'` |
| `qid` | integer not null | id вопроса в курсе |
| `block` | smallint | 1..5, только en-строки |
| `sec` | text | только en-строки |
| `q` | text not null | |
| `opts` | jsonb not null | порядок канонический |
| `correct` | smallint | только en-строки |
| `re` | text | объяснение |

RLS: одна политика `bank: owner read` — SELECT только пользователю с
**активной** строкой `user_courses` этого курса. Политик записи нет: загрузка
seed'ов и синки контента — только service role. Индекс
`bank_questions_course_lang_qid` под пагинацию `.range()` плеера.

### `public.course_trials`
**Два определения в репозитории** (оба `create table if not exists`):

- `supabase/sql/subscriptions-schema.sql` (2026-07-18): колонки `user_id`,
  `course_id`, `device`, `email_norm`, `created_at`; PK (user_id, course_id);
  индексы по (course_id, device) и (course_id, email_norm); RLS включён, политик
  нет (только сервер).
- `supabase/sql/trial-3day.sql` (2026-08-01): колонки `user_id` (FK →
  auth.users), `course_id`, `started_at`; PK (user_id, course_id); RLS + política
  `trials: own select` (кабинет читает историю триалов).

Какое определение фактически применилось в живой БД и есть ли обе группы
колонок — UNKNOWN (зависит от порядка запуска; `if not exists` не изменяет
существующую таблицу). Назначение: память триалов, строки никогда не
удаляются — повторный триал невозможен (farming shield).

### `public.page_views` (`supabase/sql/page-views.sql`)
First-party аналитика (owner request 2026-07-17).

Колонки: `id` bigint identity PK, `at` timestamptz, `path` (pathname + course id
+ `?src=`-метка кампании; без токенов), `lang`, `device` (lp:device токен),
`ref` (хост реферера или "direct"; null = внутренняя навигация), `user_id`.
RLS: INSERT разрешён anon+authenticated (`with check (true)`); чтения через API
нет — только дашборд/service role. Индекс `page_views_at`.

### `public.reviews` (`supabase/reviews.sql` + `supabase/sql/reviews-translations.sql`)
Отзывы: реальные, с согласием, с модерацией.

Колонки: `id` uuid PK, `created_at`, `user_id` (on delete set null), `name`,
`exam`, `city`, `rating` int check 1–5, `body` text not null, `consent` boolean
default false, `status` check `pending|approved|rejected` default `pending`;
плюс переводы: `lang` check `en|es|ru` default `'en'`, `body_en`, `body_es`,
`body_ru` (владелец добавляет переводы при одобрении).
RLS: insert — только своя строка и только со status `pending` (самопубликация
невозможна); select — только `approved` (anon+authenticated); UPDATE/DELETE
политик нет — модерация только через дашборд/service role.

### `public.support_tickets` (`supabase/support_tickets.sql`)
Тикеты из чат-виджета.

Колонки: `id` uuid PK, `created_at`, `updated_at`, `kind` check
`question|request|complaint`, `status` check `new|in_progress|done|rejected`
default `new`, `email` not null, `locale`, `user_id` (set null), `course_id`,
`target_lang`, `message` not null, `admin_note`.
Индексы по status/email/user_id. Триггер `support_tickets_touch` (before
update) обновляет `updated_at` через `touch_updated_at()`.
RLS: insert — все (anon+authenticated, публичная форма); select — только свои
(authenticated); UPDATE/DELETE политик нет — статусы меняет только service role.

### `public.social_stats` (`supabase/sql/report-kpi.sql`)
Счётчики соцсетей (сейчас — подписчики Telegram; FB/IG «later when Meta API is
wired»). Колонки: `day` date, `network` text, `value` int; PK (day, network).
RLS включён, политик нет — только service role (пишет `daily-stats`).

## Функции БД

| Функция | Файл | Сигнатура и поведение |
|---|---|---|
| `register_device(p_token text, p_confirm boolean=false)` | `supabase/devices_anti_sharing.sql` | security definer; возвраты: `'ok'`, `'add'` (есть место, спросить подтверждение), `'swap'` (лимит, но смена доступна), `'blocked'` (лимит + 30-дневное окно), `'unauthenticated'`, `'error'` (токен < 8 симв.). Лимит `lim := 5`, окно `win := 30 days`; при swap вытесняется устройство с самым старым `last_seen_at`; тестеры (`is_tester`) — без лимита и без диалога (owner decision 2026-08-01). Grant: только `authenticated` |
| `start_trial(p_course text)` | `supabase/sql/trial-3day.sql` | security definer; проверяет вход в список 14 платных курсов → `'not_paid_course'`; одна запись в `course_trials` навсегда → `'trial_used'`; живой доступ → `'already_active'`; иначе вставляет trial-строку и upsert в `user_courses` со `status='active'`, `expires_at = now()+3 days` → `'ok'`. Grant: только `authenticated` |
| `daily_signups(days int=8)` | `supabase/sql/report-kpi.sql` | security definer (читает `auth.users`); регистрации по дням Pacific; revoke от public/anon/authenticated — только service role |
| `recent_signups(days int=40)` | `supabase/sql/report-kpi.sql` | user_id + день регистрации; для атрибуции каналов в daily-stats; доступ как выше |
| `touch_updated_at()` | `supabase/support_tickets.sql` | trigger-функция `updated_at = now()` |
| `notify_ticket_email()` | `supabase/ticket_email_trigger.sql` | security definer; `pg_net.http_post` → Edge Function `ticket-email` на INSERT и на UPDATE со сменой статуса на done/rejected; заголовок `x-ticket-secret` (плейсхолдер `__TICKET_WEBHOOK_SECRET__` в файле) |
| `notify_ticket_issue()` | `supabase/ticket_issue_trigger.sql` | security definer; `pg_net.http_post` → Edge Function `ticket-issue` на INSERT тикетов kind `request|complaint` |

## Триггеры

| Триггер | Таблица | Событие | Функция |
|---|---|---|---|
| `support_tickets_touch` | support_tickets | before update | `touch_updated_at()` |
| `support_tickets_email` | support_tickets | after insert or update | `notify_ticket_email()` |
| `support_tickets_issue` | support_tickets | after insert | `notify_ticket_issue()` |

## Cron-задания (pg_cron)

| Job | Расписание | Действие | Файл |
|---|---|---|---|
| `expire-subscriptions` | `30 7 * * *` (00:30 PT) | `user_courses.status='inactive'` для строк со `status in ('active','trial')` и истёкшим `expires_at` | `supabase/sql/subscriptions-schema.sql` §3 |
| `daily-stats` | `0 15 * * *` (08:00 LA летом) | `net.http_post` → Edge Function `daily-stats` (или через Dashboard-Cron — рекомендованный путь в файле) | `supabase/sql/cron-daily-stats.sql` |

Расширения, включаемые скриптами: `pg_cron`, `pg_net`
(`cron-daily-stats.sql`, `ticket_*_trigger.sql`).

## Storage

Бакет `support-uploads` (`supabase/support_uploads_storage.sql`): приватный,
лимит 2 097 152 байт, mime `image/jpeg|png|webp`; файлы по пути
`<auth.uid()>/<uuid>.jpg`; RLS на `storage.objects`: insert/select только в
своей папке. Назначение: скриншоты в чат саппорта, модель получает
time-limited signed URL.

## Данные, добавляемые скриптами (не схема)

- Бета-экспирация платных строк на 2026-08-01 (`subscriptions-schema.sql`,
  `stripe-payments.sql` A2/B2).
- Бэкфилл гайда `contractor-business` всем существующим аккаунтам
  (`supabase/sql/backfill-business-guide.sql`, owner decision 2026-07-19).
- Продление платного доступа 5 тестерским аккаунтам до 2026-11-01
  (`supabase/sql/extend-tester-trials-oct31.sql`, owner instruction 2026-07-28;
  email-адреса перечислены в файле — в базу знаний не переносятся: публичный
  репозиторий, персональные данные).
- Восстановление бета-политик самозаписи до Aug 1
  (`supabase/sql/restore-beta-policies-until-aug1.sql`).
- Seed платных банков: `supabase/sql/seed-*.sql` и `bank_questions.csv` —
  генерируются (`scripts/generate-bank-*.js`) и в git не попадают (`.gitignore`).

## Зафиксированные расхождения между файлами (факты)

- Комментарий в `docs/schema.sql` о devices говорит «max 3»;
  действующий лимит — 5 (`devices_anti_sharing.sql`, owner decision 2026-07-20,
  «reverted from 3»).
- `course_trials` определена по-разному в двух файлах (см. выше); фактическая
  схема живой таблицы — UNKNOWN.
- Списки платных course_id повторяются в 4+ местах SQL/функций и различаются
  по датам создания файлов (10 → 11 → 14 позиций); актуальный список из 14 —
  в `start_trial()` и `stripe-payments.sql` SECTION B.

## Source References

- `docs/schema.sql`, `docs/migration-2026-06-22-course-status.sql`
- `supabase/devices_anti_sharing.sql`
- `supabase/sql/bank-questions-schema.sql`, `subscriptions-schema.sql`,
  `stripe-payments.sql`, `trial-3day.sql`, `tester-account.sql`,
  `extend-tester-trials-oct31.sql`, `restore-beta-policies-until-aug1.sql`,
  `page-views.sql`, `report-kpi.sql`, `cron-daily-stats.sql`,
  `reviews-translations.sql`, `backfill-business-guide.sql`
- `supabase/reviews.sql`, `support_tickets.sql`, `support_uploads_storage.sql`,
  `ticket_email_trigger.sql`, `ticket_issue_trigger.sql`
- `supabase/functions/stripe-webhook/index.ts` (записи в `user_courses`),
  `.gitignore` (seed-файлы)

Все перечисленные файлы прочитаны полностью 2026-08-05.

## Verification Status

**Partially Verified.**

- `supabase/sql/license-applications.sql` (прочитан целиком 2026-08-29);
- Проверено: полное содержание всех SQL-файлов репозитория (схемы, политики,
  функции, триггеры, cron) — каждая строка таблиц выше взята из текста файлов.
- Непроверяемо из репозитория (UNKNOWN): фактическое состояние живой БД —
  какие скрипты и в каком порядке реально запущены, текущая схема
  `course_trials`, состояние cron-джобов, объём данных в таблицах.
