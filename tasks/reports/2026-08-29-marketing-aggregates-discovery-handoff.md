# REPORT — Marketing Analytics Daily Aggregates: discovery + handoff

Задача: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md` · Дата: 2026-08-29
Автор: Claude (интерактивная сессия владельца)
Private branch: `claude/marketing-analytics-aggregates` @ `5e56eef` (база — `main` @ `037bdd9`)

## Явные заявления

- **Private `main` — UNCHANGED.** Вся работа на отдельной feature-ветке.
- **Production database — UNCHANGED.** Миграция НЕ применялась.
- **Production functions/cron — NOT DEPLOYED.** Новая Edge Function не деплоилась; существующий `daily-stats` НЕ изменён ни на байт (диффа по нему нет).

## Phase 1 — Discovery (по коду приватного репозитория)

### Источники и как daily-stats считает сейчас

- **`public.page_views`** (`supabase/sql/page-views.sql`): `at, path, lang, device, ref, user_id`; RLS — public insert-only, чтения через API нет. `path` = pathname + course id + `?src=` (полные query strings и токены не пишутся уже на клиенте, `js/stats.js`). Индекс по `at`. Политики удаления/ретеншена в репо НЕТ → история копится бессрочно (ретеншен-политика живой БД — UNKNOWN).
- **Метрики email** (`supabase/functions/daily-stats/index.ts`): views = строки за PT-день; raw devices = distinct `device`; **engaged = устройства с 2+ просмотрами за день** (обоснование в коде: клиент без localStorage получает новый токен на каждую загрузку; замер 2026-08-03 — 277 из 288 устройств за 7 дней были одно-просмотровыми); signed-in = distinct `user_id`; signups — RPC `daily_signups` (SECURITY DEFINER над `auth.users`, EXECUTE отозван у public/anon/authenticated; `supabase/sql/report-kpi.sql`).
- **Channel taxonomy** (единственная в проекте): `?src=` тег побеждает (fb/facebook→facebook, ig/insta/instagram→instagram, tg/telegram→telegram, tt/tiktok→tiktok, flyer/qr→flyer, неизвестный тег→other); иначе referrer-host regex; `ref="direct"`→direct; пустой ref→internal (не источник); прочее→other. Канал устройства = первый увиденный в окне джоба; **окно 8 дней (40 в начале месяца)** — attribution window фактически 8-дневный rolling.
- **Timezone**: везде `America/Los_Angeles` (JS `toLocaleDateString("en-CA")` и SQL `at time zone`).
- **Tester exclusion**: `profiles.is_tester` — исключаются из всех user-метрик секции «Пользователи» и WAU; в device-метриках (views/devices) исключение невозможно до входа — то же ограничение в новом слое, задокументировано.
- **Cron**: `daily-stats` в 15:00 UTC ежедневно (`supabase/sql/cron-daily-stats.sql`; фактическое состояние живого cron — UNKNOWN). Идемпотентность текущего джоба: email-отправка (не пишет агрегатов), кроме `social_stats` upsert (day, network) — идемпотентен.
- **Late payment**: отдельной записи «оплата в день X» НЕ существует — `stripe-webhook` upsert'ит `user_courses` (grantOrExtend перезаписывает строку), точный день первой оплаты из схемы не восстанавливается. `trial→paid` в email — кумулятивный снепшот «(user,course) пар с trial и когда-либо Stripe-подпиской».

### Funnel-события — фактическое наличие на сервере

| Событие из задачи | Существует server-side? |
|---|---|
| roadmap started / intake completed | **НЕТ** — `license_roadmap_*` идут только в Clarity (client-side `track()`), в Supabase не пишутся |
| application assistant opened | **НЕТ** (тот же Clarity-паттерн) |
| exam-prep CTA clicked | **НЕТ** (Clarity) |
| course opened/started | **НЕТ** отдельного события; косвенно course id внутри `page_views.path` (`?id=`) |
| trial started | ЧАСТИЧНО — строка в `course_trials` (факт без точного дня в агрегируемом виде: `started_at` есть в user_courses) |
| checkout started | **НЕТ** |
| payment completed | **НЕТ** событийной записи (только состояние `user_courses`) |
| Прочее в `app_events` | только `exam_date_set`, `exam_result_reported`, `prefs_changed`, `sample_saved`, `sample_result` (`js/stats.js` lpTrack, `js/sample-quiz.js`; схема — `supabase/sql/lifecycle-notifications.sql`) |

По правилу задачи новые события НЕ добавлялись. **Follow-up plan (минимальный, отдельной задачей с privacy review):** писать в существующую `app_events` (insert-only RLS уже есть) имена `roadmap_started`, `roadmap_completed`, `course_opened {course}`, `checkout_started {course}` через существующий `lpTrack` — 4 точки вызова в текущих файлах; payment-day — писать `paid_at` при первом grant в webhook. Ничего из этого не реализовано.

### Дублирование со снепшотами

`gsc_snapshots` пишется функцией `gsc-sync` (внешние данные Google Search Console); `ga4_snapshots` / `bing_snapshots` / `clarity_snapshots` — схем в репозитории НЕТ (созданы вне репо; состав колонок — UNKNOWN). Новый слой агрегирует ТОЛЬКО first-party данные (page_views/auth/user_courses/course_trials) — пересечения с внешними снепшотами нет; сопоставление GA4-каналов с first-party taxonomy сознательно не делается.

### PII/security риски, учтённые в дизайне

- В агрегатных таблицах нет ни одной колонки с email/UUID/device/IP/referrer/пер-пользовательскими путями — только counts, day, channel, path (pathname без query).
- RLS включён, политик нет — доступ только service role (паттерн `social_stats`). Никакого `TO authenticated`, никакого SECURITY DEFINER API.
- Проценты не хранятся (вычисляются `nullif`-запросом).
- Из известных существующих рисков: `daily_signups`/`recent_signups` — SECURITY DEFINER с отозванным EXECUTE (переиспользуются, не изменялись).

## Реализация (на ветке, ничего не применено)

| Артефакт | Что делает |
|---|---|
| `supabase/sql/marketing-daily-aggregates.sql` | 3 таблицы: `marketing_daily_metrics` (day PK; page_views, unique_devices_raw, engaged_devices, signed_in_users, signups, снепшоты active_paid_subscriptions / active_trials / weekly_active_users / trial_to_paid_count, computed_at, source_version), `marketing_channel_daily` (PK day+channel; visitors, engaged_visitors, signups, views), `marketing_page_daily` (PK day+path; views, engaged_visitors). Идемпотентный DDL, RLS-enabled/no-policies, rollback в комментарии, документированный вариант read-only роли (НЕ выдан), примеры 7/30-дневных запросов |
| `supabase/functions/marketing-aggregates/index.ts` | ОТДЕЛЬНАЯ Edge Function (email изолирован полностью — fail-soft вопрос снят архитектурно): пересчитывает последние N PT-дней (default 3, clamp 1–40 — bounded backfill), upsert по PK (идемпотентно), taxonomy — verbatim-копия из daily-stats, снепшоты — те же определения, тестеры исключены из user-метрик, path → pathname |
| `scripts/check-channel-parity.js` | Ломается при расхождении двух копий taxonomy (channel taxonomy parity test) |

**Сознательно отсутствуют** (правило задачи «не выдумывать метрику»): `new_payers`/`paid_users` (нет дневного payment-события), `signups`/`course_starts`/`checkout_starts`/`payments` в page_daily (нет подтверждённой event-связки page→signup), time/location, произвольные query strings.

## Verification

| Проверка | Результат |
|---|---|
| Channel taxonomy parity | **PASS** — блоки в daily-stats и marketing-aggregates байт-идентичны (23 строки) |
| Поведенческий тест классификатора (по реальному тексту кода): src-тег побеждает, qr→flyer, unknown→other, direct, internal→null, fb/ig/t.me/tiktok referrer, search→other | **13/13 PASS** |
| Нормализация path (query отрезается, пустой→"?") | PASS (в составе 13) |
| Синтаксис TS (node --experimental-strip-types --check) | PASS |
| `node scripts/verify.js` (весь репо) | 0 ошибок, 138 JS OK |
| Дифф `daily-stats/index.ts` | **пуст** — email-отчёт и получатели не менялись |
| SQL применение / idempotency-на-БД / timezone-boundary / RLS-grant на живой БД / Supabase advisors | **BLOCKER — не выполнялись**: у сессии нет доступа к Supabase (ни ключей, ни branch database); тестов на production не проводилось (правило задачи). Выполнить при review/после approve на branch/dev окружении |

## Rollback

`drop table` трёх таблиц (в файле миграции); исходные данные не затрагиваются — слой производный, пересоздаётся backfill'ом.

## Нерешённые вопросы атрибуции

1. Канал→оплата: невозможно без payment-day записи (follow-up в webhook).
2. Attribution window 8 дней: регистрация позже 8 дней после первого визита получает канал «—» (наследовано от email; для агрегатов = signups без канала не распределяются по каналам, попадают только в daily.signups).
3. `signups` считает НЕ-тестеров через `recent_signups`+profiles — в отличие от KPI-строки текущего email (там `daily_signups` без фильтра тестеров); расхождение задокументировано, семантика нового слоя строже.

## Статус

**READY_FOR_REVIEW.** Merge/apply/deploy — только после APPROVED_FOR_MAIN; порядок применения при approve: (1) SQL в SQL Editor, (2) deploy `marketing-aggregates`, (3) cron 30 15 * * *, (4) ручной вызов с `{"days": 30}` для первичного backfill.
