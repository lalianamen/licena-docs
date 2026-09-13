# TASK — Полная аналитика LICENA: общий план

Последняя сверка: 2026-09-11

## Status

**Этап 1: DONE — код в production, серверные шаги выполнены владельцем 2026-09-11 21:32–21:35 PT**
(скриншоты владельца): SQL применён — `select roadmap_started, roadmaps_saved, cohort_prev_signups
from public.marketing_weekly_funnel('2026-09-07')` вернул `0 · 1 · 11` (новые колонки есть; 1 план
сохранён на неделе с 07.09, 11 аккаунтов на неделе 31.08–06.09); `daily-stats` передеплоен
(страница функции: «a few seconds ago», код 719 строк — как в репозитории); вызов с телом
`{"week":"2026-09-07"}` ответил `weekly:true` от новой версии. Содержимое нового письма
(блоки Roadmap / экзамен / подписки / удержание / источники / Google) владельцем на момент
записи не подтверждено — UNKNOWN. Попутно: тест с телом по умолчанию (`{"name":"Functions"}`)
отправил обычное дневное письмо (`weekly:false`) — функция по-прежнему не проверяет роль
вызывающего.

Ранее: CODE RELEASED — `main` осн. репо @ `8bd3699` (fast-forward, 2026-09-12, команда
владельца «приступаем»); ждёт шагов владельца в Supabase (Runbook ниже). GitHub Pages отдаёт
`stats.js?v=5` + `app-roadmap.js?v=26` на `roadmap.html`; байты `app-roadmap.js`,
`app-application.js`, `app-course.js` в production совпадают с репозиторием — клиентские
события roadmap / application / exam уже пишутся в `app_events` и GA4. Серверная часть
(новые колонки SQL, секции письма) появится после шагов 1–2 Runbook.

Ранее: CODE READY_FOR_REVIEW — ветка @ `8bd3699` (2026-09-11).

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

## Runbook — шаги владельца после мержа в `main` (выполнены 2026-09-11, см. Status)

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

### Этап 2 — настройки GA4 и раскрытие — DONE (2026-09-12)

`privacy.html` выложен: `main` осн. репо @ `55ebec5` (fast-forward, команда владельца
«Заливаем», 2026-09-12); GitHub Pages отдаёт страницу с девятью упоминаниями «Google Analytics 4»
(3 языка × §1/§3/§4) и датой «September 12, 2026», байты совпадают с репозиторием.
Настройки GA4 выполнены владельцем 2026-09-11 (ниже).

Выполнено владельцем (скриншоты GA4, проперти «Licena», ресурс `properties/551663895`):
- **Специальные параметры** (Admin → Просмотр данных → Специальные определения): пять
  параметров области «Событие» — `classification`, `course`, `lang`, `page`, `state`
  (параметр события = имя), созданы 11 сент. 2026.
- **Ключевые события** (Admin → Просмотр данных → События → вкладка «Ключевые события»):
  `account_created`, `checkout_started`, `checkout_completed`, `roadmap_questionnaire_completed`
  плюс стандартное `purchase` — 5 строк. В этой версии интерфейса кнопки «Новое ключевое
  событие» нет, а звёздочкой можно пометить только уже пришедшее событие; поэтому четыре
  события созданы через Google Analytics Admin API (`properties.keyEvents.create`, API
  Explorer в документации, `countingMethod: ONCE_PER_EVENT`; первый ответ —
  `properties/551663895/keyEvents/15763906729`, `createTime 2026-09-12T05:39:31Z`).
- Поправка к плану: событие `purchase` в GA4 не приходит — покупку пишет только
  `stripe-webhook` в `app_events`; аналог покупки в GA4 — `checkout_completed` (возврат из
  Stripe). Стандартное `purchase` в списке ключевых остаётся без данных («Поток данных не
  обнаружен»).
- Хранение данных (14 месяцев) — необязательный шаг, выполнен ли — UNKNOWN.

Ранее: IN_PROGRESS (2026-09-12, команда владельца «приступаем»).

**Часть Claude — `privacy.html` (выложено, `main` @ `55ebec5`):**
`privacy.html` (одна страница, три языковых блока EN/ES/RU) до правки НЕ упоминала GA4 (grep
`Google Analytics|_ga` — 0 совпадений; факт из `14_ANALYTICS.md`, дополнение 2026-08-26).
Добавлено в каждом блоке: §1 «Information we collect» — Google Analytics 4 (Google LLC) считает
просмотры страниц, источники трафика и события продукта в агрегированном виде, Google —
поставщик услуг; §3 «Service providers» — пункт Google Analytics 4; §4 «Cookies» — first-party
cookies `_ga`, `_ga_*`; дата «Last updated» → 12 сентября 2026. Ничего не утверждается о
настройках проперти (Google signals, анонимизация IP) — они в репозитории не видны.
Проверка: Playwright — переключение EN/RU/ES, в видимом блоке ровно три упоминания GA4 и
строка `_ga, _ga_*`, дата обновлена; desktop и 360px без горизонтального переполнения и без
ошибок консоли (кроме внешнего ресурса, блокируемого прокси песочницы) — 6/6 + 2/2.

**Часть владельца — в интерфейсе GA4 (проперти «Licena Web», `G-1YE5GDRVFZ`), ~15 мин:**

1. Admin (шестерёнка) → Data display → **Custom definitions** → Create custom dimension.
   Пять раз, Scope = Event: Dimension name `state`, Event parameter `state`; затем
   `classification`, `course`, `page`, `lang` (имя = параметр). Данные копятся с момента
   создания, задним числом не пересчитываются.
2. Admin → Data display → **Key events** → New key event → ввести имя события → Save:
   `account_created`, `roadmap_questionnaire_completed`, `checkout_started`. Событие `purchase`
   GA4 считает ключевым по умолчанию — проверить, что оно в списке.
3. Необязательно: Admin → Data collection and modification → **Data retention** → Event data
   retention = 14 months (по умолчанию 2 месяца; влияет только на Explorations).

Проверка после шага 1: через сутки в Reports → Engagement → Events выбрать
`roadmap_started` — в карточках параметров появятся `state` и `lang`.

### Этап 3 — соцсети через API платформ — DONE (2026-09-12; `main` @ `1c2a038`, передеплоено владельцем, второй запуск 07:18 UTC)

Второй запуск после передеплоя (`1c2a038`): `ok:true`, `fetched_at 2026-09-12T07:18:44Z`, те же
8 снимков (facebook/posts 11, instagram/media 10, page_insights_7d 28, page_insights_28d 112),
подписчики instagram 8 / facebook 0, одно предупреждение —
`post.post_impressions_unique: (#100) The value must be a valid insights metric`: из пяти
кандидатов метрик постов Meta принимает четыре (`post_clicks`, `post_reactions_by_type_total`,
`post_video_views`, `post_activity_by_action_type`); одна отвергнутая проверяется раз за запуск
и остаётся в `warnings` как факт, не ошибка. Ежедневный cron (jobid 19, 14:20 UTC) работает без
участия владельца. Открытое: замена Page-токена (истекает 2026-11-11) на токен системного
пользователя Business Manager (`META_PAGE_ID` в коде готов); TikTok — вручную.

Ранее: LIVE, правка @ `1c2a038` выложена, ждала передеплоя (2026-09-12).

Выполнено владельцем 2026-09-11 23:00 – 2026-09-12 00:10 PT (скриншоты и ответы):
- Meta-приложение `Licena Analytics` (ID `1398512172435649`, тип «Компания», режим разработки;
  старое `LICENA Publisher` создано под один сценарий и полного списка разрешений в Explorer
  не показывало). Токен пользователя с `pages_show_list`, `pages_read_engagement`, `read_insights`,
  `instagram_basic`, `instagram_manage_insights` (+ `business_management`), продлён в Access Token
  Debugger; `me/accounts` вернул пустой список (у аккаунта нет классической роли на странице —
  доступ через бизнес-портфолио), поэтому Page-токен получен выбором страницы в Graph API
  Explorer. Отладчик: тип Page, страница Licena `1266303576558220`, Instagram `licena_us`
  `17841444125961351`, **истекает 2026-11-11 06:24 UTC** (около 60 дней; доступ к данным —
  2026-12-11). Бессрочный вариант — токен системного пользователя Business Manager (в коде
  с `1c2a038` есть `META_PAGE_ID` для него) — отдельным шагом до 11 ноября.
- Supabase: секрет `META_PAGE_TOKEN`, `social-snapshots.sql` выполнен, `meta-sync` и
  `daily-stats` задеплоены, cron `meta-sync` создан (jobid 19, `20 14 * * *`).
- Первый запуск (curl с сервисным ключом; тестовая панель Supabase даёт 403 — она шлёт публичный
  ключ): `ok:true`, `fetched_at 2026-09-12T07:08:08Z`, окна 2026-09-05…09-11 и 08-15…09-11,
  снимки `facebook/account 1`, `instagram/account 1`, `facebook/page_insights_7d 28`,
  `facebook/page_insights_28d 112`, `facebook/posts 11`, `instagram/account_insights_7d 1`,
  `instagram/account_insights_28d 1`, `instagram/media 10`; `followers`: instagram 8, facebook 0.
  Instagram — без предупреждений (все метрики аккаунта и медиа приняты). Facebook отверг
  `page_impressions`, `page_impressions_unique`, `post_impressions` («(#100) The value must be a
  valid insights metric» — Meta отключила метрики показов 2026-06-15), принял
  `page_post_engagements`, `page_daily_follows_unique`, `page_daily_unfollows_unique`,
  `page_video_views` (28 строк = 4 метрики × 7 дней).
- Правка `1c2a038` (ветка): `FB_PAGE_METRICS` без метрик показов; метрики постов
  (`post_impressions_unique`, `post_clicks`, `post_reactions_by_type_total`, `post_video_views`,
  `post_activity_by_action_type`) проверяются один раз за запуск на первом посте, принятые
  используются для остальных; `totals()` суммирует объекты-разбивки; `META_PAGE_ID`;
  строка Facebook в письме — вовлечения · новых подписчиков · отписок · просмотров видео.
  Проверки: TS-синтаксис, `test-meta-core` 16/16, verify 139. После мержа — повторный деплой
  `meta-sync` и `daily-stats`.
- Замечание к проверке письма: снимок `account_insights_7d` от 12.09 покрывает 05.09–11.09, а
  не отчётную неделю с понедельника 07.09, поэтому блок «Instagram за неделю» появится с первого
  понедельничного запуска cron (14.09 14:20 UTC); подписчики и таблица постов доступны сразу.

Ранее: CODE RELEASED (`main` @ `47f2a2f`, 2026-09-12); CODE READY_FOR_REVIEW (ветка @ `47f2a2f`).

Ранее: CODE READY_FOR_REVIEW (2026-09-12, ветка @ `47f2a2f`).

Ответ владельца 2026-09-11 (скриншоты): Instagram `@licena_us` (8 подписчиков) и страница
Facebook «Licena» (0 подписчиков) связаны в Meta Business Suite; TikTok `@licena_us` —
бизнес-аккаунт (2 подписчика, 28 лайков, 340 просмотров за 7 дней, ролики 08.09 ×2 и 11.09).
Telegram — уже в `social_stats` (подписчики, `daily-stats`).

**Сделано (ветка `claude/question-bank-generation-analysis-y47sk7` @ `47f2a2f`):**
- `supabase/functions/meta-sync/index.ts` + `core.ts` (чистое ядро): по одному долгоживущему
  Page-токену (`META_PAGE_TOKEN`) снимает и пишет в `public.social_snapshots` (platform,
  dataset, start_date, end_date, row_count, payload): `facebook/account`, `page_insights_7d`,
  `page_insights_28d` (дневные ряды `page_impressions`, `page_impressions_unique`,
  `page_post_engagements`, `page_daily_follows_unique`, `page_daily_unfollows_unique`,
  `page_video_views`), `posts` (28 дней: лайки, комментарии, репосты, `post_impressions`,
  `post_impressions_unique`, `post_clicks`); `instagram/account`, `account_insights_7d`,
  `account_insights_28d` (`reach`, `views`, `accounts_engaged`, `total_interactions`, `likes`,
  `comments`, `shares`, `saves`, `profile_links_taps` как `total_value`), `media` (посты и Reels за
  28 дней: `like_count`, `comments_count`, `view_count`; insights `views`, `reach`, `saved`,
  `shares`, `total_interactions`, для Reels ещё `ig_reels_avg_watch_time`,
  `ig_reels_video_view_total_time`). Метка `?src=…` из подписи сохраняется (`src`). Недоступная
  метрика не роняет синхронизацию — попадает в `warnings` ответа (страницы Facebook до 100
  лайков Page Insights не получают — по документации Meta). Подписчики Instagram / Facebook
  за день — upsert в `social_stats` (network `instagram` / `facebook`). Сториз не снимаются
  (живут 24 часа). Названия метрик — по справочникам Meta, прочитанным 2026-09-12 (`impressions`
  удалён с v22, заменён `views`/`reach`).
- `supabase/sql/social-snapshots.sql` (таблица, RLS без политик, revoke anon/authenticated),
  `supabase/sql/cron-meta-sync.sql` (ежедневно `20 14 * * *` UTC — за 40 минут до письма; по
  понедельникам 7-дневное окно = ровно отчётная неделя Пн–Вс).
- `daily-stats`: дневное письмо — строки Instagram / Facebook рядом с Telegram (дельта к
  предыдущему сохранённому дню); недельная секция — блок «Соцсети»: подписчики с дельтой за
  неделю, итоги Instagram за неделю (охват, просмотры, вовлечённые аккаунты, взаимодействия,
  переходы по ссылкам профиля), итоги Facebook за неделю, таблица постов и роликов недели
  (площадка · дата · тип · просмотры · охват · лайки · комментарии · сохранения · репосты ·
  метка · визитов на сайт по метке · ссылка на пост), итог за 28 дней.
- `scripts/test-meta-core.mjs` — 15 тестов ядра (окна дат: понедельник → Пн–Вс прошлой недели;
  наборы метрик по поверхности; разбор `total_value` / рядов / lifetime; метка в подписи; окно по
  timestamp) — 15/15. TS-синтаксис обеих функций, verify 139, паритет каналов, core 48/48.
- НЕ проверено: живой вызов Graph API (нет токена; Deno в песочнице нет) — первый запуск
  покажет `warnings` по метрикам, которые аккаунт не отдаёт.

**Шаги владельца (после мержа):**
1. developers.facebook.com → My Apps → Create app → тип Business, название любое (например
   «Licena Analytics»), привязать бизнес-портфолио из Business Suite. Режим Development
   оставить: для собственных активов ревью не нужно.
2. Tools → Graph API Explorer: выбрать это приложение → «User or Page: Get User Access Token»
   → разрешения `pages_show_list`, `pages_read_engagement`, `read_insights`, `instagram_basic`,
   `instagram_manage_insights` → Generate → в окне Meta выбрать страницу Licena и Instagram.
3. Tools → Access Token Debugger: вставить токен → Debug → кнопка «Extend Access Token» →
   скопировать долгоживущий токен пользователя (60 дней).
4. В Graph API Explorer запрос `GET /me/accounts?fields=name,access_token` с долгоживущим токеном
   → в ответе у страницы Licena поле `access_token` — это Page-токен без срока действия
   (пока пользователь не сменит пароль / не отзовёт приложение).
5. Supabase: `supabase secrets set META_PAGE_TOKEN=<page-token>` (или Edge Functions → Secrets);
   SQL Editor → `supabase/sql/social-snapshots.sql`; `supabase functions deploy meta-sync`;
   `supabase functions deploy daily-stats`; SQL Editor → `supabase/sql/cron-meta-sync.sql`
   с подставленными `<PROJECT_REF>` и `<SERVICE_ROLE_KEY>`.
6. Проверка: вызвать `meta-sync` (POST, пустое тело, сервисный ключ, как `bing-sync`) — в
   ответе `ok:true`, `snapshots` (8 строк), `followers`, `warnings`; затем `daily-stats` с телом
   `{"week":"<понедельник>"}` — в письме блок «Соцсети».

**TikTok** — API отдаёт только данные авторизованного аккаунта через приложение TikTok for
Developers (Display API: `video.list` с просмотрами, лайками, комментариями, репостами) после
ревью приложения; до ревью — sandbox, отдаёт ли он реальные данные своего аккаунта — UNKNOWN.
Решение: пока вручную (лист «Соцсети»); функция `tiktok-sync` — отдельной задачей, если
владелец заведёт приложение.

Ранее: TODO, не начат.

| Платформа | Что даёт API | Что нужно от владельца | Решение |
|---|---|---|---|
| Instagram / Facebook (Meta Graph API) | по посту и Reels: охват, просмотры, среднее время просмотра, лайки, комментарии, сохранения, репосты; подписчики | Instagram Business/Creator, привязка к Facebook-странице, приложение в Meta for Developers (режим разработки, без ревью для своего аккаунта), токен раз в 60 дней | ждёт ответа владельца, на каких площадках ведётся публикация |
| TikTok | по видео: просмотры, лайки, комментарии, репосты (без времени просмотра) | приложение в TikTok for Developers + ревью доступа | вручную раз в неделю до получения доступа |
| Telegram (просмотры постов) | только клиентский API (MTProto) от имени аккаунта | сессия аккаунта на сервере | не делать (риск для аккаунта) |
| YouTube | просмотры / лайки / комментарии по видео | ключ API | только если канал ведётся — UNKNOWN |

Механика при реализации: Edge Function `meta-sync` → таблица снимков (как `bing_snapshots`)
→ строки в понедельничном письме; cron.

### Этап 4 — дашборд вместо переноса цифр — CODE READY_FOR_REVIEW (2026-09-13, ветка `claude/question-bank-generation-analysis-y47sk7`, не в `main`)

**Сделано (Claude):** `supabase/sql/reporting-looker.sql` — схема `reporting` и роль
`licena_reporter` (login, пароль подставляет владелец, `statement_timeout` 60 s), которая
читает ТОЛЬКО представления этой схемы; в `public` и `auth` прав нет (проверено на локальном
PostgreSQL: `permission denied`). Ни одно представление не отдаёт user_id, e-mail, device-токен
или сырой просмотр. Представления:

| Представление | Источник |
|---|---|
| `daily_metrics`, `channel_daily`, `page_daily`, `state_snapshots` | таблицы `marketing_*` (без служебных колонок) |
| `social_followers` | `social_stats` (день · сеть · подписчики) |
| `events_daily` | `app_events` — день (PT) · имя события · событий · людей |
| `weeks` | последние 26 полных PT-недель (Пн) |
| `weekly_funnel`, `weekly_roadmaps`, `weekly_sources` | три недельные функции через `lateral` по `weeks`; `weekly_sources` отдаёт kind · метка `?src=` · referrer · центы |
| `bing_daily` | новейший снимок `rank_traffic` |
| `gsc_daily`, `gsc_queries` | новейшие снимки `date` и `query,page` в `gsc_snapshots`; колонки jsonb/timestamp находятся по типу (таблица создана вне репо), при их отсутствии представления пропускаются с notice |
| `social_posts` | новейшие `media` / `posts` из `social_snapshots`: площадка · дата · тип · метка · текст · ссылка · просмотры · охват · лайки · комментарии · сохранения · репосты |
| `instagram_windows` | итоги аккаунта Instagram за 7 и 28 дней по датам окна |

Прогон на локальном PostgreSQL 16 с макетом всех таблиц и функций: 12 представлений
читаются ролью (`gsc_*` созданы динамически), выборки совпали с ожидаемыми (атрибуция покупки к
метке `ig-…`, Reel с меткой из подписи, GSC/Bing по дням), сырые `public.page_views`,
`public.marketing_daily_metrics`, `auth.users` — `permission denied`. На живой базе не
выполнялось.

**Шаги владельца:**
1. SQL Editor: в `supabase/sql/reporting-looker.sql` заменить `<REPORTER_PASSWORD>` на длинный
   случайный пароль → Run. Заполненный файл не сохранять. Повторный запуск безопасен.
2. Supabase → Connect → вкладка «Session pooler»: скопировать host вида
   `aws-0-<region>.pooler.supabase.com` (регион — UNKNOWN, виден там).
3. lookerstudio.google.com → Create → Data source → **PostgreSQL** → host из п. 2, port `5432`,
   database `postgres`, username `licena_reporter.vewhmndummfhnbxnrqya`, пароль из п. 1,
   «Enable SSL» → Authenticate → выбрать представление (или Custom query) → Connect.
   Одному источнику соответствует одно представление; основные: `weekly_funnel`,
   `daily_metrics`, `channel_daily`, `events_daily`, `social_posts`, `gsc_daily`, `bing_daily`.
4. Страницы отчёта (предложение; строится в интерфейсе Looker Studio владельцем или Claude при
   доступе): «Обзор» (KPI за 7 дней из `daily_metrics`, ряд просмотров/устройств/регистраций),
   «Воронка по неделям» (`weekly_funnel` таблицей, конверсии как вычисляемые поля),
   «Источники» (`channel_daily`, `weekly_sources` по меткам), «Продукт» (`events_daily`,
   `weekly_roadmaps`), «SEO» (`gsc_daily`, `gsc_queries`, `bing_daily`), «Соцсети»
   (`social_followers`, `social_posts`, `instagram_windows`), «Деньги» (`state_snapshots`).

Ранее: TODO.

### Дополнительно (2026-09-13, та же ветка): `daily-stats` только для сервисной роли

`daily-stats` принимала любой валидный JWT, включая публичный ключ, и каждый вызов отправлял
письмо (владелец дважды получал лишние письма из тестовой панели). Добавлена та же проверка,
что в `bing-sync` / `meta-sync`: сервисный ключ (legacy или `sb_secret_…` через admin-пробу),
иначе 403. Cron `daily-stats` уже шлёт сервисный ключ — не затронут. После мержа — деплой
`daily-stats`; тестовая панель Supabase перестанет срабатывать, проверка — curl с сервисным
ключом.

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
