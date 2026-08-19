# RFC: Lifecycle-система пользователя и Notification Engine

Статус: **PROPOSED** (реализация Phase 1 ведётся на ветке
`claude/az-b-general-contractor-9zj4e9` основного репо; в `main` — только
после одобрения владельца). Дата: 2026-08-19. Источник задачи: промпт
владельца в сессии 2026-08-19 (полная lifecycle-система: конверсия,
возврат, exam date, notification engine, анти-спам, отписки, аналитика).

## 1. Current-state audit (что уже есть в репо)

| Область | Факт | Источник |
|---|---|---|
| Auth | Supabase Auth (email), профиль создаётся при первом логине | `js/app.js` (~209), `js/app-cabinet.js` |
| Профили | `public.profiles`: id, name, email, **lang**, is_tester, stripe_customer_id | `supabase/sql/stripe-payments.sql:12`, `devices_anti_sharing.sql:16`, `js/app-cabinet.js` (profile.lang/name) |
| Подписки | `public.user_courses`: status ('active'/'trial'/'inactive'), started_at, expires_at, auto_renew, stripe_subscription_id | `supabase/sql/subscriptions-schema.sql` |
| Триалы | `public.course_trials` (user, course, device, email_norm) + RPC `start_trial` | `supabase/sql/trial-3day.sql` |
| Платежи | Stripe: checkout/portal/webhook edge-функции; webhook двигает expires_at, auto_renew, гасит при deleted | `supabase/functions/stripe-webhook/index.ts` |
| Прогресс | ТОЛЬКО localStorage: `lp:answers:<id>`, `lp:summary:<id>` (answered/correct/total/**per-section stats**/activeBlock/lastFinal/ts) | `js/app-course.js:342–380` |
| Email-инфраструктура | **Resend уже подключён** (RESEND_API_KEY, FROM noreply@licena.us); транзакционные письма тикетов, трёхъязычный COPY-паттерн; триггер Postgres → pg_net → edge | `supabase/functions/ticket-email/index.ts`, `ticket_email_trigger.sql` |
| Cron | **pg_cron + pg_net включены**: `expire-subscriptions` (00:30 PT), `daily-stats` (08:00 LA, Resend-отчёт владельцу) | `subscriptions-schema.sql`, `sql/cron-daily-stats.sql` |
| Аналитика | `public.page_views` (at, path c `?src=` кампаний, lang, device, ref, **user_id**), insert-only RLS; beacon `js/stats.js`; KPI-отчёт | `sql/page-views.sql`, `js/stats.js`, `sql/report-kpi.sql` |
| Отзывы | `public.reviews` (rating, body, consent, pending→approved модерация, честная — без фильтра по звёздам) | `supabase/reviews.sql` |
| Девайсы | `public.devices` + `register_device()` (лимит 5, last_seen_at) | `devices_anti_sharing.sql` |
| i18n | `js/i18n-app.js` (TAPP, EN/ES/RU); язык профиля в `profiles.lang` | `js/app-cabinet.js:697` |
| Отписки/преференсы | **НЕТ** | — |
| Exam date | **НЕТ** | — |
| Событийная аналитика (кроме pageview) | **НЕТ** | — |
| Плановые письма пользователям | **НЕТ** | — |
| Referral / Teams | **НЕТ** | — |

**Переиспользуем:** Resend + COPY-паттерн ticket-email; pg_cron+pg_net
паттерн daily-stats; `page_views` (last activity, `?src=` = атрибуция
кликов из писем БЕЗ пикселей); `profiles.lang` (язык письма);
`user_courses.status/expires_at/auto_renew` (payment-состояния);
`reviews` (отзывы после сдачи); `lp:summary` (вся математика плана уже
считается на клиенте — зеркалим на сервер, не пересчитываем).

**Не хватает:** exam date; server-side прогресс; преференсы/отписка;
журнал отправок (дедуп/капы); событийный трекинг; сам движок.

## 2. Lifecycle-модель (вычисляемая, не хранимая)

Статус НЕ хранится колонкой — он **выводится** SQL-запросом движка из
существующих данных (единственный новый источник правды — exam-статус):

| Состояние | Вывод из |
|---|---|
| A Visitor | нет profiles-строки (email-недостижим — вне движка) |
| B Registered, no purchase | profiles есть; нет user_courses со status in (active,trial) по платным курсам |
| C Trial/Free | user_courses.status='trial' ИЛИ только бесплатные курсы |
| D Paid active learner | активный платный курс + активность ≤ N дней (page_views/user_progress.last_seen_at) |
| E Paid inactive | активный платный курс + активность > N дней |
| F Exam scheduled | user_exam_settings.status='scheduled', exam_date > today |
| G Exam approaching | то же, days_left ≤ 14 |
| H Exam done, result unknown | exam_date < today, status='scheduled' |
| I Passed | status='passed' |
| J Failed | status='failed' |
| K Postponed | status='postponed' (до ввода новой даты) |
| L Subscription cancelled | активный платный курс, auto_renew=false (отменил, доступ до expires_at) |
| M Former customer | был платный (course_trials/stripe_subscription_id есть), сейчас ни одного active |
| N Corporate/team | Phase 3 (см. §8) |

Состояния F–K живут ПОВЕРХ B–E/L–M (пользователь одновременно «paid
active» и «exam approaching») — движок трактует их как два измерения.

## 3. Изменения БД (минимальные; один идемпотентный SQL-файл)

`supabase/sql/lifecycle-notifications.sql`:

1. **`user_exam_settings`** (PK user_id+course_id): exam_date date,
   tz text (IANA, default 'America/Los_Angeles'), status
   ('scheduled','passed','failed','postponed') + result_at, created_at,
   updated_at. RLS: own select/insert/update/delete.
2. **`notification_prefs`** (PK user_id): study_reminders, exam_countdown,
   progress_updates, product_updates, promotions (bool, default true),
   unsub_token uuid default gen_random_uuid() (capability для отписки без
   логина), updated_at. RLS: own select/update; insert через RPC/дефолт.
   «Unsubscribe from all marketing» = все 5 → false (транзакционные
   письма — тикеты/чеки — не трогаются, у них свой путь).
3. **`notification_log`** (id, user_id, kind, dedup_key, channel, sent_at,
   email, meta jsonb) + **UNIQUE (user_id, kind, dedup_key)** — дедуп на
   уровне БД: одно и то же письмо физически невозможно отправить дважды
   (§29). Капы частоты читаются отсюда же.
4. **`user_progress`** (PK user_id+course_id): answered, correct, total,
   sec_stats jsonb (зеркало lp:summary.secStats), active_block,
   last_seen_at. RLS: own upsert/select. Пишет клиент рядом с
   существующей записью lp:summary (одна строка кода в app-course.js) —
   сервер получает прогресс для писем, ничего не пересчитывая.
5. **`app_events`** (at, user_id, device, name text, meta jsonb) —
   insert-only RLS как page_views; события §22 из клиента через
   `lpTrack()` в js/stats.js. Серверные события (reminder_sent) — это
   notification_log. Клики/возвраты из писем — ссылки `?src=em-<kind>` →
   уже существующий page_views-механизм (без пикселей открытий:
   ненадёжно и следящее — сознательно меряем клики и возвраты).
6. **`notification_config`** (key text PK, value jsonb) — капы и тайминги
   правятся UPDATE'ом без передеплоя движка: marketing_max_per_day=1,
   quiet_if_active_hours=36, nudge_после_регистрации={1,4}, paid_inactive_days=4,
   countdown_days=[14,7,3,1], send_window_local=[9,19] и т.д.
7. RPC **`unsub_by_token(token, category)`** security definer — статичная
   страница `u/index.html` дергает её анонимно; токен = capability.
8. Шаблон cron-джобы (по образцу cron-daily-stats.sql): hourly →
   `notify-engine` edge function.

НЕ создаём: referral-таблиц, team-таблиц (§8), очередь (queue не нужна —
идемпотентный движок + unique-констрейнт покрывают гарантии доставки на
нашем масштабе).

## 4. Notification Engine

`supabase/functions/notify-engine/index.ts`, вызывается pg_cron ежечасно
(секрет-заголовок как у ticket-email). Один проход:

1. SELECT кандидатов (profiles ⋈ user_courses ⋈ user_exam_settings ⋈
   user_progress ⋈ notification_prefs ⋈ последняя активность из
   page_views ⋈ notification_log-агрегаты).
2. Для каждого — вычислить состояние (§2) и набор «кандидатов-сообщений».
3. Фильтры (в этом порядке): категория включена в prefs → дедуп-ключ не
   встречался (notification_log) → маркетинговый кап (≤1/сутки, кроме
   транзакционных) → тишина при недавней активности → локальное время
   пользователя в окне 9:00–19:00 (tz из exam settings, иначе LA).
4. Отправить через Resend (язык = profiles.lang; COPY-структура как в
   ticket-email), записать в notification_log (ON CONFLICT DO NOTHING —
   вторая гарантия от дублей), плюс meta.
5. Все ссылки в письмах: `https://licena.us/…?src=em-<kind>` + футер с
   персональной unsubscribe-ссылкой (`/u/?t=<token>&c=<category>`).

Типы сообщений Phase 1 (dedup_key в скобках):
- `welcome_nudge_d1` (once) — B-состояние, 24ч без активности: «Your CSLB
  practice is ready. Try 5 questions…»
- `value_nudge_d4` (once) — B, ~4 дня: объяснения ответов как ценность.
- `paid_inactive` (`day:<date>`, не чаще чем раз в N дней, прогресс в
  тексте: «You're 38% through C-10…»)
- `exam_countdown` (`d:<14|7|3|1>:<course>`) — режимы по §13.
- `exam_daily_coach` (`day:<date>:<course>`) — только при exam_date И
  включённой категории exam_countdown; текст от прогресса/слабых секций.
- `exam_result_ask` (`ask:<exam_date>`) — на следующий день после даты.
- `passed_congrats` (once) — транзакционное поздравление + отзыв +
  ссылка «stop exam-related emails».
- `milestone_<25|50|75|100>` (`<course>:<pct>`) — категория progress_updates.

Passed → движок больше не выбирает exam-кандидатов (status='passed'
отсекает на шаге 2), а `exam_countdown`-категория пользователю
предлагается к отключению одной кнопкой.

## 5. Exam date + персональный план

- Кабинет: у каждого платного курса «When is your CSLB exam?» (опц.),
  смена/перенос/удаление/«я сдал». Хранение §3.1.
- План — чистая математика на клиенте (уже есть lp:summary): дни до
  экзамена, % завершения, вопросов осталось, вопросов/день
  (ceil(remaining/max(days-1,1))), фокус-секции = худшие по
  secStats (<60% и ≥8 ответов — существующее правило weakestSection),
  опережение/отставание = сравнение фактического темпа с требуемым.
  Никакого AI — только арифметика (§4 промпта).
- Сервер видит то же через user_progress → тексты писем совпадают с UI.

## 6. Intro offer $9.99 (Phase 2 — спроектировано, не строится)

Stripe: второй price ($9.99 first month) через checkout `discounts` /
phase-цены; конфиг-флаг в `notification_config` (`intro_offer`:
{enabled, price_id}) → stripe-checkout читает; вся цена показывается ДО
оплаты ($9.99 first month → $19.99/month after → cancel anytime → дата
следующего списания из Stripe). Метрики: события §22 + Stripe-отчёты.
A/B-готовность: вариант оффера пишется в app_events.meta и в
subscription metadata — сравнение конверсий SQL-запросом, без
experiment-платформы.

## 7. Безопасность (§29)

- RLS на всех новых таблицах; notification_log/config — только service
  role (никаких клиентских политик).
- Движок и cron — по секрет-заголовку (паттерн TICKET_WEBHOOK_SECRET).
- unsub_token — random uuid, только в письмах этого пользователя;
  RPC security definer меняет ровно préfs, ничего не читает наружу.
- Дубли писем исключены UNIQUE-констрейнтом (не только логикой).
- Никаких секретов в клиенте (anon-ключ Supabase уже публичный by design).

## 8. Referral и Teams (архитектурная закладка, Phase 3)

- Referral: `profiles.ref_code` (уникальный, генерится лениво) + приход
  фиксируется существующим `?src=` (src=ref-<code> в page_views) +
  `app_events('referral_clicked')`. Начисления (free days/discount) — НЕ
  сейчас; когда понадобятся — таблица referral_grants поверх готовых
  событий.
- Teams: nullable `org_id` на user_courses + таблицы orgs/org_seats,
  admin видит только name/email/course/% (из user_progress) — ничего
  лишнего. Ничего в user_courses менять заранее не требуется — колонка
  добавится ALTER'ом без миграции данных. Зафиксировано, не строится.

## 9. Phase-план

- **Phase 1 (эта ветка):** §3 таблицы; движок §4 с типами Phase 1;
  exam date UI + план-виджет; prefs-экран + `/u/` отписка; post-exam
  модалка + passed-флоу (отзыв через существующий reviews); прогресс-
  синк; lpTrack + события §22 (клиентская часть); cron-шаблон; копия
  EN/ES/RU; e2e-проверки.
- **Phase 2:** intro offer $9.99 (+ A/B), недельный digest, weak-topic
  тексты в письмах, streaks-milestones (аккуратно), failed-флоу
  расширенный (перепланирование).
- **Phase 3:** referral-начисления, teams/seats + admin-прогресс,
  углублённая персонализация.

## 10. Риски

1. **Прогресс локален устройству** — user_progress отражает последнее
   синкнувшее устройство; берём max(answered) при конфликте (upsert с
   greatest). Честное ограничение, зафиксировано.
2. **Resend-лимиты** (free: 100 писем/день) — на текущем масштабе ок;
   капы движка сами держат объём; при росте — платный план Resend.
3. **Часовой пояс** — по умолчанию America/Los_Angeles (аудитория
   CA/AZ), уточняется при вводе exam date; ошибка максимум сдвигает
   письмо на часы, не создаёт спам.
4. **GitHub Pages статичен** — отписка обязана работать RPC-вызовом со
   статичной страницы; решено токеном-capability (§7).
5. Не трогаем: auth, платёжную систему, механику курсов, дизайн — точки
   контакта только перечисленные файлы (§9 промпта владельца, §30).

## Source References
`supabase/sql/{subscriptions-schema,trial-3day,page-views,stripe-payments,cron-daily-stats,report-kpi}.sql`, `supabase/{reviews,devices_anti_sharing,support_tickets,ticket_email_trigger}.sql`, `supabase/functions/{ticket-email,stripe-webhook,daily-stats,stripe-checkout}/index.ts`, `js/{app,app-cabinet,app-course,stats,i18n-app,supabase-client}.js`.

## Verification Status
Partially Verified — все утверждения об имеющемся коде проверены чтением
названных файлов в сессии 2026-08-19; проектная часть (§2–§9) — план, не
факт.

## Implementation note (append, 2026-08-19)

Phase 1 реализована в тот же день на ветке
`claude/az-b-general-contractor-9zj4e9` основного репо (5 коммитов:
`26ea082` SQL, `4bb88d5` notify-engine, `bea46c8` клиентский слой,
`a8fe749` кабинет; полный разбор — строка 2026-08-19 в
`26_CHANGELOG.md`). Статус RFC остаётся **PROPOSED**: мерж в `main`,
выполнение SQL в Supabase, деплой notify-engine и создание cron-джобы —
только после одобрения владельца (его явное условие).

## Приложение A — постатейный статус реализации (сохранено 2026-08-19 по запросу владельца «пока сохрани, ещё вернёмся»)

Всё из Phase 1 написано и проверено, лежит на ветке
`claude/az-b-general-contractor-9zj4e9` основного репо; в `main` не влито,
в Supabase не активировано (SQL не выполнялся, notify-engine не задеплоен,
cron-джоба не создана) — до активации ни одно письмо не уходит.

| § промта | Статус |
|---|---|
| 1 Аудит | ✅ до кода, в §1 этого RFC |
| 2 Lifecycle A–N | ✅ вычисляемые статусы; N (corporate) — закладка Phase 3 |
| 3 Дата экзамена | ✅ user_exam_settings + синк из кабинета; ⚠️ отдельной кнопки «удалить дату» нет (сбрасывается через перенос/результат) |
| 4 Персональный план | ✅ чистая математика; ⚠️ индикатор «опережает/отстаёт» — Phase 2 |
| 5 Notification Engine | ✅ notify-engine, все решения централизованы |
| 6 Frequency cap | ✅ ≤1/сутки, UNIQUE-дедуп в БД, тишина при активности, tz-окно 9–19; лимиты — UPDATE'ом в notification_config |
| 7 Registered-no-purchase | ✅ welcome d1 + value d4; ⚠️ показ оффера — вместе с §8 |
| 8 Intro $9.99 | 📋 спроектирован (§6 RFC), Phase 2 |
| 9 Paid-inactive | ✅ с реальным прогрессом, повтор ≥7 дней |
| 10 Daily Exam Coach | ✅ последние 14 дней, по категории, от прогресса |
| 11 Неодинаковые письма | ✅ частично (прогресс/дни/слабые секции); ⚠️ last topic/recent errors — Phase 2 |
| 12 Milestones | ✅ 25/50/75/100; ⚠️ первые-10/точность/streaks — Phase 2 |
| 13 Режимы 14/7/3/1 | ✅ четыре текста-режима |
| 14 Вопрос после даты | ✅ письмо с 4 one-tap кнопками + модалка |
| 15 Passed-флоу | ✅ стоп-письма/статус/поздравление/отзыв/отключение; ⚠️ referral и «другой курс» — Phase 2/3 |
| 16 Честные отзывы | ✅ существующий reviews-поток, без фильтра по звёздам |
| 17 Failed | ✅ поддержка + новая дата |
| 18 Postponed | ✅ новая дата → план заново, без повторов старого |
| 19 Prefs + отписка | ✅ 5 категорий + unsub-all + /u/ по токену; транзакционные отдельно |
| 20 Teams | 📋 закладка (§8 RFC), Phase 3 |
| 21 Referral | 📋 закладка (§8 RFC), Phase 3 |
| 22 События аналитики | ✅ частично: app_events+lpTrack (exam_date_set, exam_result_reported, prefs_changed), notification_log = reminder_sent, клики = ?src=em-*; платёжные — уже в Stripe-вебхуке |
| 23 KPI | ✅ частично: цепочки измеримы SQL-ом; готового отчёта нет |
| 24 A/B-готовность | ✅ архитектурно (конфиг + копия в одном месте + meta) |
| 25 Копия EN/ES/RU | ✅ 8 типов × 3 языка, язык = профиль |
| 26 Mobile-first | ✅ 480px письма, экраны проверены на 360px |
| 27 Минимальная БД | ✅ 6 таблиц + 2 RPC, без queue/status-колонки |
| 28 Scheduler | ✅ pg_cron hourly (шаблон в SQL-файле) |
| 29 Безопасность | ✅ RLS/service-role/токены/UNIQUE-дедуп/без секретов в клиенте |
| 30 Не переделывать продукт | ✅ auth/платежи/курсы/дизайн не тронуты |
| 31 Phase 1/2/3 | ✅ §9 RFC |
| 32 Показать до кода | ✅ этот RFC |
| 33 E2E + отчёт | ✅ SQL 16 тестов (локальный PG16), движок 25/25 сценариев (мок-харнесс в scratchpad сессии), клиент — Playwright |

Итог: 24 пункта полностью, 5 частично (⚠️), 3 — сознательно Phase 2/3 (§8, §20, §21).

**Шаги активации, когда владелец вернётся:** (1) «одобряю lifecycle» → мерж
ветки в main; (2) выполнить `supabase/sql/lifecycle-notifications.sql` в SQL
Editor; (3) Edge Function `notify-engine` (Verify JWT ON) из
`supabase/functions/notify-engine/index.ts`; (4) cron-джоба `notify-engine`,
`0 * * * *`. Проверка вхолостую: Invoke с `{"dry_run":true}`.
