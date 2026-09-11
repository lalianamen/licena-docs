# TASK — Полная аналитика LICENA: общий план

Последняя сверка: 2026-09-11

## Status

**IN_PROGRESS — Этап 1 в работе** (2026-09-11, команда владельца «собери это в общий план, и начинай»).

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
| 1.1 | События roadmap и Application Assistant в `app_events` + GA4: `roadmap_started`, `roadmap_questionnaire_completed`, `roadmap_viewed` (раз на загрузку), `roadmap_step_completed` {step}, `roadmap_practice_clicked`, `application_started`, `application_ready`, `application_submitted`; параметры `state`, `classification`, `lang`. Clarity-имена не меняются. На 4 страницах roadmap/application бикон `pageview.js` заменяется на `stats.js` (те же строки `page_views`, но с `user_id` вошедшего и с `lpTrack`) | `js/roadmap/app-roadmap.js`, `js/roadmap/app-application.js`, `roadmap*.html`, `application.html` | в работе |
| 1.2 | Строки roadmap в недельном письме: начали → завершили анкету → открыли план → отметили шаг → перешли в практику → Application Assistant (начали / пакет собран / отметили «подано»); сохранённых планов за неделю по штату и классификации (`license_roadmaps`, без тестеров) | `marketing-weekly-funnel.sql`, `daily-stats` | в работе |
| 1.3 | Атрибуция за неделю по первому визиту: таблица «канал · метка · новых устройств · аккаунтов · покупок · выручка» (связка `purchase.user_id` → устройства пользователя → первый просмотр → `?src=` / referrer) | новая SQL-функция `marketing_weekly_sources`, `daily-stats` | в работе |
| 1.4 | Google Search Console в письме: показы / клики за дни недели из последнего снимка `gsc_snapshots` (датасет `date`) и топ-5 запросов по кликам (датасет `query,page`); строка «визиты из ИИ-ассистентов» (канал `ai`) | `daily-stats` | в работе |
| 1.5 | `exam_completed` {course, pct, pass, correct, total, timed_out} при завершении экзамена-симуляции; в письме «завершили экзамен / сдали» | `js/app-course.js` (`gradeExam`), SQL, `daily-stats` | в работе |
| 1.6 | Подписки в недельной секции: активные оплаченные на конец недели и изменение за неделю, активные триалы (из `marketing_state_snapshots`), окончившихся подписок (`subscription_ended`) | SQL, `daily-stats` | в работе |
| 1.7 | Когорты: аккаунты прошлой недели, из них вернулись на этой; аккаунты 4 недели назад, из них вернулись на этой; «вернулись из письма» (устройства с `?src=em-…`, метка ставится `notify-engine`) | SQL, `daily-stats` | в работе |

Шаги владельца после мержа (runbook будет в карточке этапа): (1) выполнить обновлённый
`supabase/sql/marketing-weekly-funnel.sql` в SQL Editor (файл начинается с `drop function`,
потому что набор колонок меняется); (2) `supabase functions deploy daily-stats`;
(3) проверка: `daily-stats` с телом `{"week":"<PT-понедельник>"}`.

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

- `lalianamen/llicena@main`: `js/stats.js`, `js/pageview.js`, `js/ga.js`, `js/app-course.js`,
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

**Partially Verified** — раздел «Что уже есть» проверен чтением названных файлов; оценки объёма
работ по этапам 2–4 и возможности API платформ — из ответов Claude владельцу, по документации
платформ не перепроверялись в этой сессии; отмеченные `UNKNOWN` места — как есть.
