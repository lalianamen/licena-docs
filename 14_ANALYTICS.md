# 14 — Аналитика и сбор событий

Последняя сверка: 2026-08-05 (полная) · 2026-08-11 (точечная: секция «Пользователи») · 2026-08-26 (точечная: GA4) · 2026-08-30 (точечная: маркетинговые агрегаты)

Дополнение 2026-08-12 (`72fc7a5` осн. репо): подключён Microsoft Clarity
(проект «LICENA», Project ID `y1ic13wlta`, аккаунт владельца) — тепловые
карты и записи сессий; загрузчик `js/clarity.js` (внешний файл из-за CSP без
'unsafe-inline', no-op при недоступности clarity.ms), тег на 123 страницах
(index/app/course, practice/guides, /about/; tools/privacy/terms — без
записи). Clarity ставит first-party cookie `_clck`/`_clsk` — раскрыто в
`privacy.html` §1/§3/§4 ×3 языка. First-party бикон `js/stats.js` →
`page_views` не менялся и работает параллельно.
Источник: `lalianamen/llicena@main`. Только first-party; сторонних трекеров
нет (grep gtag/googletagmanager/analytics.js по HTML и `js/*.js` — пусто).
Числа — в `15_METRICS.md`.
(Факт «сторонних трекеров нет» действителен по 2026-08-25 — с 2026-08-26
подключён GA4, см. дополнение ниже.)

Дополнение 2026-08-26 (`fca5fb2` осн. репо): подключён Google Analytics 4
(задача владельца; Measurement ID `G-1YE5GDRVFZ`, GA-проперти «Licena Web»
в аккаунте владельца). Схема установки: официальный загрузчик
`<script async src="https://www.googletagmanager.com/gtag/js?id=G-1YE5GDRVFZ">`
литерально в `<head>` каждой страницы + конфиг-сниппет вынесен в `js/ga.js`
(`?v=1`; внешний файл, потому что CSP сайта не содержит `'unsafe-inline'`
для скриптов — тот же паттерн, что `js/clarity.js`). Покрытие: все 171
обслуживаемые страницы (root + `/es` + `/ru`: index, about, app, course,
preview, practice-preview, 60 practice, 57 guides, 30 tools, demo/cabinet,
privacy, terms, `/u`); вставка сразу после CSP-меты (на 4 страницах без
CSP-меты — demo/cabinet, privacy, terms, `/u` — после `<meta charset>`).
CSP на 167 страницах расширен по официальному CSP-гайду Google для Google
tag: `script-src` +`https://*.googletagmanager.com`; `img-src`
+`https://*.google-analytics.com` +`https://*.googletagmanager.com`;
`connect-src` +`https://*.google-analytics.com`
+`https://*.analytics.google.com` +`https://*.googletagmanager.com`.
НЕ тронуты: email-шаблоны (`docs/email/`, `supabase/email-templates/`),
печатные флаеры (`docs/marketing/flyer/`), файл верификации
`googlec4665304f2eceeb0.html`. Перед установкой grep подтвердил отсутствие
GA4/GTM на сайте — экземпляр тега один. События GA4 — стандартные
автоматические (page_view и enhanced measurement по настройкам проперти);
кастомные события в gtag не отправляются — событийная кастом-аналитика
остаётся на Clarity (`track()` в `js/sample-quiz.js`) и first-party биконе.
`privacy.html` на 2026-08-26 НЕ упоминает cookies GA4 (`_ga`, `_ga_*`) —
раскрытие за владельцем (для Clarity `_clck`/`_clsk` раскрытие было сделано
2026-08-12).

## Событие — ровно одно: page view

Один fire-and-forget INSERT в `public.page_views` на загрузку страницы.
Поля записи (оба бикона держат одинаковую форму):

| Поле | Что пишется | Чего НЕ пишется |
|---|---|---|
| `path` | pathname + `id=<course>` + `src=<метка>`; `utm_source` принимается как алиас `src`; метка обрезается до 40 симв., фильтр `[^\w-]` | полный query string, токены |
| `lang` | язык UI из `lp:ui_lang` | — |
| `device` | первопартийный токен `lp:device` (crypto.randomUUID, создаётся и для анонимов) | cookies (не используются), IP (биконом не передаётся) |
| `ref` | ХОСТ реферера без `www.` (обрезка 60 симв.); пустой реферер → `"direct"`; свой хост → `null` (внутренняя навигация) | полный referrer-URL |
| `user_id` | id сессии, если вошёл | — |

Сбой аналитики никогда не ломает страницу — всё в try/catch, «an analytics
failure can't break anything» (`js/stats.js`, шапка).

## Два бикона

- `js/stats.js` (48 строк, прочитан полностью) — index/app/course; через
  supabase-js (`supa.from("page_views").insert`), с определением user_id.
- `js/pageview.js` (48 строк, полностью) — публичные `/practice/*`; сырой
  REST-fetch к PostgREST БЕЗ загрузки supabase-js («SEO landing pages that
  must stay light»); анонимный (`user_id: null`), `keepalive: true`,
  path обрезается до 200 симв.

Хранение: `page_views` — RLS insert-only (anon+authenticated); чтения через
API нет — только дашборд/service role (`supabase/sql/page-views.sql`).

## Кампании и каналы

Источник визита размечается меткой `?src=` в ссылках (выживает в in-app
браузерах Instagram/TikTok/Telegram, которые срезают referrer) с fallback на
хост реферера. Классификатор в `daily-stats`
(`supabase/functions/daily-stats/index.ts:31–53`):

- метки: `fb|facebook→facebook`, `ig|insta|instagram→instagram`,
  `tg|telegram→telegram`, `tt|tiktok→tiktok`, `flyer|qr→flyer`,
  неизвестная → `other`;
- referrer-регэкспы: facebook/fb.com/fb.me, instagram, t.me|telegram,
  tiktok; прочее (включая Google) → `other`; `direct` — без реферера;
  `null` — внутренняя навигация (не источник).

## Ежедневный отчёт (`daily-stats`, 265 строк, прочитан полностью)

- Запуск: cron `0 15 * * *` UTC (08:00 Los Angeles летом) —
  `supabase/sql/cron-daily-stats.sql`; чтение service role'ом.
- Окно: 8 дней; 1-го числа месяца окно расширяется до 40 дней и добавляется
  месячный rollup (границу месяца определяет сама ежедневная джоба).
- Состав письма: вчерашние числа; таблица «Канал | Пришли | Регистр. | KPI»
  (порядок строк — owner request 2026-07-18: Прямая ссылка, Флаер/QR,
  Facebook, Instagram, Telegram, TikTok, Прочее (поиск и др.)); 7-дневная
  таблица; топ страниц. Язык отчёта — русский (таблицы/месяцы в коде).
- Воронка per-channel (`sourceStats`): канал устройства = канал ПЕРВОГО
  визита; «зарегистрировался» = устройство связано (любым залогиненным
  визитом) с user_id, созданным в окне отчёта; KPI = регистрации/визиты %.
  Регистрации — из `daily_signups`/`recent_signups` (security definer по
  `auth.users`, Pacific-дни — `supabase/sql/report-kpi.sql`).
- Масштабная оговорка в коде: сырые строки за окно «fine at our scale;
  revisit if views exceed ~50k».
- Доставка: Resend, получатели `STATS_EMAIL` (дефолт в коде — личный адрес,
  в базу знаний не переносится).

## Счётчики соцсетей

`public.social_stats (day, network, value)` — пишет `daily-stats` service
role'ом; сейчас — подписчики Telegram (секрет `TELEGRAM_BOT_TOKEN` в
функции); «FB/IG later when Meta API is wired»
(`supabase/sql/report-kpi.sql`). Живые значения — UNKNOWN.

## Внешняя аналитика

Google Search Console: property верифицирована 2026-07-14
(`googlec4665304f2eceeb0.html`; `docs/marketing/gsc-readout-2026-08.md`);
первый экспорт снят 2026-08-05 (числа — `15_METRICS.md`). Других внешних
систем аналитики в репозитории нет.

## UNKNOWN

- Живые объёмы `page_views` / `social_stats`, фактические KPI из писем.
- Работает ли cron в живой БД (файл — рекомендация Dashboard-Cron).
- Серверные логи Supabase/GitHub Pages (IP и пр.) — вне репозитория.

## Дополнение 2026-08-11: секция «Пользователи» в daily-stats

По запросу владельца («более подробный отчёт по пользователям для полной
картины») ежедневное письмо `daily-stats` дополнено секцией «Пользователи»
(git `main` осн. репо `4bf702f`, функция `usersSection` в
`supabase/functions/daily-stats/index.ts`). Источники данных: `auth.users`
через admin API (email, created_at, last_sign_in_at), `profiles` (lang,
is_tester), `user_courses` (status, stripe_subscription_id, expires_at,
auto_renew, activated_at), `course_trials`, `devices`, окно `page_views`.
Состав: итоги аккаунтов (тестеры помечаются 🎁 и исключаются из сводных
цифр), «ни разу не входили», новые (вчера/7/30 дней), активные по входам
(вчера/7 дней), оплаченные подписки (+автопродление), активные триалы,
истёкшие строки, конверсия триал→оплата, свод по курсам и две таблицы —
«активны вчера» (email, просмотры, открытые курсы, устройства) и «новые
вчера» (email, язык, канал прихода, добавленные курсы), максимум 30 строк.
В письме явно указано: поимённый прогресс по вопросам хранится в
localStorage пользователя и на сервере НЕ существует. Fail-soft: при сбое
секции базовый отчёт посещений всё равно отправляется. Деплой — ручной
(владелец вставляет обновлённый файл в Edge Function; новых секретов и SQL
не требуется). Статус на 2026-08-11: код в репозитории, задеплоен ли в
Supabase — UNKNOWN (за владельцем).

## Дополнение 2026-08-30: слой ежедневных маркетинговых агрегатов

Задача `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`. Отдельный от письма
слой аналитики: Edge Function `marketing-aggregates` (ветка
`main` @ `ae13124`, мерж 2026-08-30)
раз в сутки в 15:30 UTC пересчитывает последние 3 завершённых Pacific-дня и
пишет обезличенные агрегаты в 4 таблицы —
`marketing_daily_metrics` (день: page_views, unique_devices_raw,
engaged_devices, signed_in_users, signups, weekly_active_users),
`marketing_channel_daily` (день × канал: visitors, engaged_visitors, signups,
views), `marketing_page_daily` (день × путь: views, engaged_visitors),
`marketing_state_snapshots` (день запуска: active_paid_subscriptions,
active_trials, trial_to_paid_count). Схема и RLS — `04_DATABASE.md`,
устройство функции — `06_FUNCTIONS.md`.

Отличия от ежедневного письма (оба слоя читают один и тот же `page_views`):
- письмо остаётся источником оперативной картины и НЕ изменялось (диффа в
  `supabase/functions/daily-stats/index.ts` нет);
- агрегаты хранят историю в БД, поэтому пригодны для запросов за 7/30 дней
  без пересборки писем;
- «вовлечённое устройство» = ≥ 2 просмотра за день; окно атрибуции канала
  устройства здесь фиксированное — 28 дней до конца целевого дня (у письма —
  скользящее 8-дневное), что делает пересчёт любого дня детерминированным;
- таксономия каналов побайтово совпадает с письмом
  (`scripts/check-channel-parity.js`);
- тестеры (`profiles.is_tester`) исключены из пользовательских метрик; на
  уровне устройств до входа фильтрация невозможна — то же ограничение, что у
  письма.

Персональных данных слой не хранит: только счётчики по дню/каналу/пути.

Фактическое состояние на 2026-08-30 (подтверждено выводом Supabase Dashboard,
`tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`): таблицы
созданы, функция задеплоена, выполнен backfill за 30 завершённых PT-дней
(2026-07-30 … 2026-08-28) — 30 строк в `marketing_daily_metrics`, 62 в
`marketing_channel_daily`, 491 в `marketing_page_daily`; повторный
идентичный запуск не создал дублей; cron-задача зарегистрирована и активна.
Первый автоматический запуск по расписанию на дату сверки ещё не состоялся
(UNKNOWN).

## Source References

- `js/stats.js`, `js/pageview.js` — полностью
- `supabase/functions/daily-stats/index.ts` — полностью;
  `supabase/sql/page-views.sql`, `report-kpi.sql`, `cron-daily-stats.sql`
- `googlec4665304f2eceeb0.html`, `docs/marketing/gsc-readout-2026-08.md`
- grep-проверка отсутствия сторонних трекеров (index/app/course + `js/*.js`)
- `js/ga.js`, `index.html` (образец вставки GA4 + расширенного CSP),
  коммит `fca5fb2` осн. репо (список всех 171+167 затронутых страниц)
- `supabase/functions/marketing-aggregates/core.ts` и `index.ts`,
  `supabase/sql/marketing-daily-aggregates.sql`,
  `supabase/sql/cron-marketing-aggregates.sql`,
  `scripts/check-channel-parity.js` (ветка
  `main` @ `ae13124`)
- `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`,
  `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`

## Verification Status

**Verified** (по коду) с оговоркой: всё о живых данных и работе cron —
UNKNOWN (раздел выше); сами механизмы подтверждены полным чтением обоих
биконов, функции отчёта и SQL.
