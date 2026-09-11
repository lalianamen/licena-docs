# TASK — Полная аналитика LICENA: общий план

Последняя сверка: 2026-09-11

## Status

**Этап 1: CODE READY_FOR_REVIEW — ветка `claude/question-bank-generation-analysis-y47sk7`
@ `8bd3699`, не в `main`, не задеплоено** (2026-09-11). Мерж — по команде владельца; после мержа
два шага владельца в Supabase (Runbook ниже).

Ранее: IN_PROGRESS (2026-09-11, команда владельца «собери это в общий план, и начинай»).

Контекст: воронка трекера `Licena_All_Metrics_Tracker.xlsx` реализована и подтверждена в
production (`tasks/ANALYTICS_FUNNEL_TRACKER.md`, DONE). Этот документ собирает в один план
всё, чего для «полной аналитики» ещё нет, по слоям и этапам, с тем, что делает Claude, и тем,
что требует шагов владельца.

## Определение

«Полная аналитика» = каждый шаг пути пользователя (увидел → попробовал → зарегистрировался →
учится → купил → остался → вернулся) считается автоматически, в одном месте, с разрезом по
источнику (канал / метка поста), штату и классификации.

## Что уже есть (факты по репозиторию `lalianamen/llicena@main`, 2026-09-11)

| Слой | Реализовано | Источник |
|---|---|---|
| Просмотры страниц | first-party `page_views` (`js/stats.js` на index/app/course; лёгкий бикон `js/pageview.js` на practice-, roadmap- и application-страницах), GA4 `page_view`, `?src=` → `campaign_source` в GA4 | `js/stats.js`, `js/pageview.js`, `js/ga.js` v2 |
| События воронки | 17 именованных событий в `app_events` + GA4 (hero_cta_click … purchase / subscription_*) | `tasks/ANALYTICS_FUNNEL_TRACKER.md` §1 |
| Каналы и метки постов | классификатор `<канал>-<что угодно>` в `daily-stats` и `marketing-aggregates/core.ts`; дневные агрегаты `marketing_channel_daily` | `supabase/functions/daily-stats/index.ts`, `supabase/functions/marketing-aggregates/core.ts` |
| Недельная воронка | SQL `marketing_weekly_funnel` + понедельничная секция письма (счётчики, конверсии, основной источник, Bing) | `supabase/sql/marketing-weekly-funnel.sql`, `daily-stats` |
| Roadmap / Application Assistant | 15 событий только в Clarity; планы вошедших пользователей в `license_roadmaps` / `license_roadmap_steps`; просмотры страниц через `pageview.js` (user_id всегда null) | `js/roadmap/app-roadmap.js` (`track()` строка 42), `js/roadmap/app-application.js` (`track()` строка 32), `supabase/sql/license-roadmaps.sql` |
| Деньги | `marketing_state_snapshots` (активные оплаченные подписки, триалы, trial→paid по дням); покупки и выручка за неделю в письме | `supabase/sql/marketing-daily-aggregates.sql`, `marketing-weekly-funnel.sql` |
| Удержание | D1/D7 — приближения по устройствам; WAU в письме | `marketing-weekly-funnel.sql`, `daily-stats` |
| SEO | `gsc-sync` → `gsc_snapshots` (5 датасетов, 28 дней), `bing-sync` → `bing_snapshots` (5 датасетов); в письме только Bing-строка | `supabase/functions/gsc-sync/index.ts`, `supabase/functions/bing-sync/index.ts` |
| ИИ-видимость | канал `ai` в классификаторе (chatgpt / openai / perplexity / claude.ai / gemini / copilot по referrer) | `daily-stats/index.ts` `channelOf` |
| Соцсети | подписчики Telegram ежедневно в `social_stats` и письме; переходы по `?src=` | `daily-stats` (TELEGRAM_BOT_TOKEN), `marketing_channel_daily` |
| Единое место | письмо `daily-stats` + ручной перенос в xlsx | `daily-stats` |

## Этапы

### Этап 1 — один цикл деплоя (Claude; затем два шага владельца) — IN_PROGRESS

Всё, что требует одинаковых шагов владельца (выполнить SQL, передеплоить `daily-stats`),
собрано в один этап, чтобы деплой был один.

| # | Что | Где | Статус |
|---|---|---|---|
| 1.1 | События roadmap и Application Assistant в `app_events` + GA4: `roadmap_started`, `roadmap_questionnaire_completed`, `roadmap_viewed` (раз на загрузку), `roadmap_step_completed` {step}, `roadmap_practice_clicked`, `application_started`, `application_ready`, `application_submitted`; параметры `state`, `classification`, `lang`. Clarity-имена не меняются; demo-сценарии (`?demo=`) не считаются. На 4 страницах roadmap/application бикон `pageview.js` заменён на `stats.js` (те же строки `page_views`, но с `user_id` вошедшего и с `lpTrack`) | `js/roadmap/app-roadmap.js` v26 (`FUNNEL`, `track(name, meta)`), `js/roadmap/app-application.js` v7, `roadmap.html`, `roadmap-az.html`, `roadmap-nv.html`, `application.html` | готово (ветка) |
| 1.2 | Строки roadmap в недельном письме: начали → завершили анкету → открыли план → отметили шаг → перешли в практику → Application Assistant (начали / пакет собран / отметили «подано»); конверсии «анкета → план», «план → практика»; сохранённых планов за неделю всего и по штату × классификации (`license_roadmaps`, без тестеров) | `marketing-weekly-funnel.sql` (колонки `roadmap_*`, `application_*`, `roadmaps_saved`; функция `marketing_weekly_roadmaps`), `daily-stats` (`roadmapsWeekHtml`) | готово (ветка) |
| 1.3 | Атрибуция за неделю по первому визиту: таблица «канал · метка · новых устройств · аккаунтов · покупок · выручка» (`purchase.user_id` → все устройства пользователя → самый ранний первый просмотр → `?src=` / referrer через тот же классификатор `channelOf`; человек без просмотров — строка «Неизвестно») | функция `marketing_weekly_sources` (kind = device / account / purchase), `daily-stats` (`sourcesWeekHtml`) | готово (ветка) |
| 1.4 | Google Search Console в письме: показы / клики / CTR за дни недели из новейшего снимка `gsc_snapshots` (датасет `date`; колонка payload находится по форме `{dimensions, rows}`, новейший снимок — по `end_date`; указано, сколько дней недели покрыто) и топ-5 запросов по кликам за окно снимка (датасет `query,page`); строка «визиты из ИИ-ассистентов» (устройства с входным каналом `ai`) | `daily-stats` (`gscWeek`, `entryChannels`) | готово (ветка) |
| 1.5 | `exam_completed` {course, pct, pass, correct, total, timed_out} в `gradeExam` при выставлении результата экзамена-симуляции; в письме «завершили экзамен · сдали (%)» | `js/app-course.js` v75, SQL (`exam_completed`, `exam_passed`), `daily-stats` | готово (ветка; в браузере не прогонялось — страница курса требует входа, проверен синтаксис и область видимости `courseId`) |
| 1.6 | Подписки в недельной секции: оплаченных активных на последний день недели (последний снимок `marketing_state_snapshots` ≤ воскресенья, с датой снимка) и изменение за неделю, активных триалов, окончившихся подписок за неделю (`subscription_ended`, уникальные события Stripe) | SQL (`subscriptions_ended`), `daily-stats` (`subsWeek`) | готово (ветка) |
| 1.7 | Когорты: аккаунты прошлой недели → сколько из них имели просмотр со входом на этой неделе; то же для аккаунтов недели 4 недели назад; «вернулись из письма» — устройства с `?src=em-…` (метка ставится `notify-engine`) | SQL (`cohort_prev_*`, `cohort_4w_*`, `email_returns`), `daily-stats` | готово (ветка) |

Что НЕ вошло в этап 1 (осознанно): `license_questionnaire_step_completed` и
`license_roadmap_step_opened` остаются только в Clarity (по одному событию на каждый вопрос
и каждое открытие карточки — шум для `app_events`); MRR не считается (нет источника цен по
подпискам в аналитическом слое — UNKNOWN).

## Проверки этапа 1 (на ветке @ `8bd3699`)

- `node scripts/verify.js` — 139 файлов; `check-channel-parity` OK; `test-marketing-core` 48/48;
  `test-roadmap-v3` — ALL PASS; TS-синтаксис `daily-stats` (`node --experimental-strip-types --check`).
- SQL прогнан на локальном PostgreSQL 16 с макетом таблиц (`auth.users`, `profiles`,
  `page_views`, `app_events`, `license_roadmaps`) и тестовыми данными: три функции создаются
  (`drop` + `create`), гранты применяются; счётчики совпали с ожидаемыми (дубль события Stripe
  схлопнут, тестер исключён из аккаунтов и планов, когорты 1 из 2 и 1 из 1, возврат из письма 1,
  атрибуция аккаунта и покупки к метке `ig-…` первого визита).
- Playwright (Chromium, перехват POST `app_events` / `page_views` и `dataLayer`): Аризона —
  просмотр страницы через `stats.js` с device-токеном, `roadmap_started` {state, lang} в
  `app_events` и GA4, ни одного события за время ответов, `roadmap_questionnaire_completed`
  с `classification`, `roadmap_viewed` ровно один раз, `roadmap_step_completed` со `step`,
  `roadmap_practice_clicked` доходит до `app_events` до перехода; `?demo=1` — ноль событий;
  Application Assistant — `application_started` в `app_events` и GA4; RU — `lang: ru`;
  без ошибок консоли — 15/15. 360px (roadmap RU, roadmap-nv EN, application RU): без
  горизонтального переполнения, без ошибок, события уходят — 3/3.

## Runbook — шаги владельца после мержа в `main`

1. **SQL** — Supabase → SQL Editor → вставить целиком `supabase/sql/marketing-weekly-funnel.sql`
   из `main` → Run. Файл начинается с `drop function if exists public.marketing_weekly_funnel(date)`
   (набор колонок изменился); создаёт три функции. Проверка:
   `select * from public.marketing_weekly_funnel('2026-09-07');` (должны быть колонки
   `roadmap_started … cohort_4w_returned`).
2. **Деплой** — `supabase functions deploy daily-stats` (из `main`).
3. **Проверка** — вызвать `daily-stats` с телом `{"week":"2026-09-07"}` (как 2026-09-11);
   в ответе `weekly:true`, в письме — блоки «Roadmap и Application Assistant», «Экзамен-симуляция»,
   «Подписки», «Удержание», «Источники за неделю», строки Google / Bing / ИИ-ассистенты.
   Замечание: вызов с публичным ключом отправляет письмо (функция не проверяет роль — как и раньше).
4. Тестовая строка в `app_events` (`meta.test = true`) — удалить, если ещё не удалена.

### Этап 2 — настройки GA4 и раскрытие (владелец, ~15 мин; Claude — privacy) — TODO

- В GA4 отметить как ключевые события: `account_created`, `purchase`, `roadmap_questionnaire_completed`, `checkout_started`.
- Custom dimensions (event scope): `state`, `classification`, `course`, `page`, `lang` — без них
  параметры событий не видны в стандартных отчётах GA4.
- `privacy.html` ×3 языка: раскрытие cookies GA4 (`_ga`, `_ga_*`) — UNKNOWN, раскрыто ли уже
  (проверяется при выполнении этапа).

### Этап 3 — соцсети через API платформ (по решению владельца) — TODO, не начат

| Платформа | Что даёт API | Что нужно от владельца | Решение |
|---|---|---|---|
| Instagram / Facebook (Meta Graph API) | по посту и Reels: охват, просмотры, среднее время просмотра, лайки, комментарии, сохранения, репосты; подписчики | Instagram Business/Creator, привязка к Facebook-странице, приложение в Meta for Developers (режим разработки, без ревью для своего аккаунта), токен раз в 60 дней | ждёт ответа владельца, на каких площадках ведётся публикация |
| TikTok | по видео: просмотры, лайки, комментарии, репосты (без времени просмотра) | приложение в TikTok for Developers + ревью доступа | вручную раз в неделю до получения доступа |
| Telegram (просмотры постов) | только клиентский API (MTProto) от имени аккаунта | сессия аккаунта на сервере | не делать (риск для аккаунта) |
| YouTube | просмотры / лайки / комментарии по видео | ключ API | только если канал ведётся — UNKNOWN |

Механика при реализации: Edge Function `meta-sync` → таблица снимков (как `bing_snapshots`)
→ строки в понедельничном письме; cron.

### Этап 4 — дашборд вместо переноса цифр — TODO

Looker Studio поверх read-only роли Postgres (все агрегаты уже в таблицах `marketing_*`,
`gsc_snapshots`, `bing_snapshots`, `app_events`, `social_stats`). Нужен аккаунт Google
владельца и роль с `select` только на эти таблицы (создаётся отдельным SQL, без прав на
`auth.*`, `profiles`, `user_progress`).

## Что остаётся вручную при любом этапе

- Цитирования в Bing Copilot (AI Performance) — API нет (Microsoft, 02/2026).
- Досмотры и время просмотра TikTok; всё по постам Telegram.
- Ключевые события и custom dimensions GA4 — только в интерфейсе GA4 владельцем.

## Source References

- `lalianamen/llicena` ветка `claude/question-bank-generation-analysis-y47sk7` @ `8bd3699`
  (этап 1) и `main` (раздел «Что уже есть»): `js/stats.js`, `js/pageview.js`, `js/ga.js`, `js/app-course.js`,
  `js/roadmap/app-roadmap.js`, `js/roadmap/app-application.js`, `roadmap.html`, `roadmap-az.html`,
  `roadmap-nv.html`, `application.html`, `supabase/functions/daily-stats/index.ts`,
  `supabase/functions/marketing-aggregates/core.ts`, `supabase/functions/gsc-sync/index.ts`,
  `supabase/functions/bing-sync/index.ts`, `supabase/functions/notify-engine/index.ts`,
  `supabase/sql/marketing-weekly-funnel.sql`, `supabase/sql/marketing-daily-aggregates.sql`,
  `supabase/sql/license-roadmaps.sql`
- `tasks/ANALYTICS_FUNNEL_TRACKER.md`, `14_ANALYTICS.md`
- Сообщения владельца 2026-09-11: «а родмап мы будем подключать к статистике?», «что нужно для
  полной аналитики?», «а что по социалкам?», «собери это в общий план, и начинай».

## Verification Status

**Partially Verified** — раздел «Что уже есть» и этап 1 проверены чтением названных файлов и
прогонами, перечисленными в «Проверках этапа 1» (SQL — на локальном PostgreSQL, не на живой
базе; `exam_completed` — без браузерного прогона); оценки объёма работ по этапам 2–4 и
возможности API платформ — из ответов Claude владельцу, по документации платформ не
перепроверялись в этой сессии; отмеченные `UNKNOWN` места — как есть.
