# 15 — Метрики: только реальные числа

Последняя сверка: 2026-08-05 (полная) · 2026-08-25 (точечная: аддендум NV-банков ниже) · 2026-08-30 (точечная: аддендум маркетинговых агрегатов ниже) · 2026-09-03 (точечная: аддендумы nv-b2/nv-c2 и nv-c21 ниже) · 2026-09-16 (точечная: аддендумы ориентиров CSLB, GSC и банка A ниже) · 2026-09-17 (точечная: аддендум банка la-building ниже)
Правило документа: каждое число либо ИЗМЕРЕНО в этой сверке по файлам
репозитория (помечено «репо»), либо взято из единственного внешнего
измеренного источника проекта — экспорта Google Search Console,
зафиксированного в `docs/marketing/gsc-readout-2026-08.md` (помечено «GSC»).
Предположений и оценок нет; всё неизмеренное — UNKNOWN.

## Контент (репо; пересчёт скриптом `node vm` по `js/questions/*`)

| Показатель | Значение |
|---|---|
| Банков на `content-banks-src` (полный набор) | 37 |
| EN-вопросов на `content-banks-src` | 10 350 |
| Позиций с учётом RU/ES-оверлеев | ≈ 31 050 — **вычислено как 10 350 × 3**, а не отдельным пересчётом всех локализованных записей (полнота оверлеев поштучно не сверялась) |
| Банков в `main` (бесплатные, статика) | 22 |
| EN-вопросов в `main` | 2 850 |
| Платных курсов (SKU) в продаже | 14 |
| Готовых, не запущенных банков | 1 (`la-business-law`, 500×3) |
| Цена подписки | $20/мес за курс (`stripe-checkout/index.ts:19`) |
| Триал | 3 дня, один на (user, course) (`trial-3day.sql`) |
| Гайд-курсов | 3 файла `js/guides/` |
| Сэмпл-наборов для SEO-страниц | 20 (`js/samples/`) |

## Страницы и локализация (репо)

| Показатель | Значение |
|---|---|
| URL в `sitemap.xml` | 126 (3 лендинг + 3 about + 60 practice + 57 guides + privacy + terms) |
| Статических SEO-страниц с CSP | 120 из 120 (grep 2026-08-05) |
| Ключей UI-переводов | `js/i18n.js`: 156 × 3 языка; `js/i18n-app.js`: 315 × 3 |
| Языки | EN/ES/RU + staged `hy` (1 банк, тестеры) |

## Размеры файлов (репо; `du`/`ls`, 2026-08-05)

| Объект | Размер |
|---|---|
| Весь клиентский JS `js/*.js` (с vendor) | 940 КБ |
| Vendor `supabase-js-2.110.0.js` | 204 КБ |
| CSS (4 файла) | 148 КБ |
| Каталог банков `js/questions/` (main) | 4,9 МБ |
| `index.html` | 40 807 Б |
| `app.html` | 22 815 Б |
| `course.html` | 15 309 Б |
| `og-image.png` (1200×630) | 43 570 Б |
| Пример practice-страницы (`c-10`) | 24 КБ + сэмпл-JS 28 КБ |
| Крупнейший файл банка | `backflow.ru.js` — 428 КБ |
| Файлов в репо (без .git) | 357 |

## Поисковые метрики (GSC; экспорт «last 3 months», реально
2026-07-17 → 2026-08-02, property верифицирована 2026-07-14)

Оговорка из самого отчёта: «Sample size is tiny … directional, not
statistical» — выборка мала, числа направленческие.

| Показатель | Значение (GSC) |
|---|---|
| Показы / клики за ~3 недели | 156 показов, 4 клика (все — США) |
| Брендовый запрос `licena` | 7 показов, 0 кликов, ср. позиция 8.43 |
| Лучший intent-запрос `cslb exam questions` | 7 показов, 2 клика с позиции 42.7 |
| Самая показываемая страница | `/practice/cslb-law-and-business/` — 36 показов, позиция 33 |
| `es/practice/c-27-landscaping` | позиция 1 |
| ES-лендинг | позиция 7.58, CTR 8.33% (единственный ES-клик) |
| `ru/practice/c-10-electrical` | позиция 3.75 |
| `ru/practice/cslb-law-and-business` | позиция 7.14 |
| `es/practice/c-10-electrical` | позиция 9.25 |
| Запрос `certificación epa 608` | позиция 9; `examen de leyes` — 8 |
| Конкурентные EN-запросы | `c10 license test` 62; `cslb test` 55; `general b practice test` 79 |
| Показы EPA-страниц | `ru/epa-608` 23, `es/epa-608` 14 — география международная (не Калифорния) |
| Mobile vs Desktop | ср. позиция 11.3 против 31.9 |
| Поисковые объёмы запросов | UNVERIFIED (так помечено в самом отчёте) |

## История контент-метрик (репо, датированные факты)

- 2026-07-21 → 2026-08-04: +4 платных банка по 500×3 (C-27, B-2, C-33,
  AZ SRE) — `js/bank-updates.js`.
- Аудиты банков: C-8 — 478/500 подтверждены, 2 переключены, 16 цитат
  обновлены; C-27 — 60 вопросов перецитированы на MWELO-2025, 21
  подтверждён (`js/bank-updates.js`, записи 2026-07-28).
- C-33: ключи 124/125/130/121 по A–D; B-2: ровно 125×4; C-27: 25/25/25/25 в
  каждом блоке (`js/bank-updates.js`).

## UNKNOWN (не измерено / вне репозитория)

- Пользователи: регистрации, DAU/MAU, устройства — живая БД (`page_views`,
  `auth.users`) недоступна из репозитория.
- Выручка: подписки, MRR, конверсия триал→оплата — Stripe.
- Подписчики соцсетей (Telegram/FB/IG/TT) — `social_stats` живые значения.
- Фактические KPI-письма `daily-stats` (значения таблиц).
- GSC после 2026-08-02.
- Производительность страниц (LCP и т.п.) — тестов нет
  (`16_PERFORMANCE.md`).

## Аддендум 2026-08-30 — каналы трафика после разделения `search`/`ai`

Источник: таблица `public.marketing_channel_daily` живой базы (слой
маркетинговых агрегатов, осн. репо `main` @ `c2e9dd6`). Окно — 30 завершённых
Pacific-дней **2026-07-30 … 2026-08-28**, пересчитанных бэкфиллом после
разделения бакетов. Запрос:
`select channel, sum(visitors), sum(engaged_visitors), sum(signups)
from marketing_channel_daily group by channel`. Числа приведены дословно из
результата, полученного владельцем 2026-08-30.

| channel | visitors | engaged_visitors | signups |
|---|---|---|---|
| direct | 578 | 60 | 6 |
| search | 87 | 45 | 8 |
| other | 65 | 24 | 3 |
| facebook | 2 | 0 | 0 |
| flyer | 1 | 1 | 0 |
| **итого** | **733** | **130** | **17** |

Сопоставление с состоянием ДО разделения (та же таблица, тот же период,
зафиксировано в `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`):
`other` = 152 / 69 / 11 разошёлся на `search` = 87 / 45 / 8 и `other` = 65 / 24 / 3.
Суммы по всем каналам не изменились (733 визита, 17 регистраций) — разделение
перераспределило существующие строки, а не добавило данных.

Производные величины (вычисляются из counts, в таблицах не хранятся):
- доля вовлечённых: direct 60/578, search 45/87, other 24/65;
- регистраций на визит: direct 6/578, search 8/87, other 3/65.

Ограничения этих чисел:
- окно 30 дней, абсолютные значения регистраций малы (17 всего, 8 у `search`);
- `visitors` считается по устройствам (`page_views.device`), тестеры на уровне
  устройств до входа не фильтруются — то же ограничение, что у ежедневного письма;
- бакет `ai` за это окно строк не имеет: переходов от AI-ассистентов не
  зафиксировано (бакет существует, значение — 0);
- `direct` = пустой `document.referrer` (`js/pageview.js`, `js/stats.js`), то есть
  смешанный источник: прямой ввод адреса, закладки, QR, встроенные браузеры
  приложений и любые источники, вырезающие реферер.

## Verification Status (аддендум 2026-08-30)

**Partially Verified** — числа приведены дословно из результата SQL-запроса к
живой базе, выполненного владельцем 2026-08-30 (у AI-сессии доступа к Supabase
нет); определения метрик проверены по коду
`supabase/functions/marketing-aggregates/core.ts`. Оценок и выводов документ не
содержит.

## Source References

- Пересчёты/замеры этой сверки: `js/questions/*` (обе ветки), `sitemap.xml`,
  `js/i18n*.js`, `du`/`ls` по перечисленным файлам, grep CSP по 120
  страницам
- `docs/marketing/gsc-readout-2026-08.md` — полностью (все строки «GSC»)
- `js/bank-updates.js`, `supabase/functions/stripe-checkout/index.ts`,
  `supabase/sql/trial-3day.sql`

## Verification Status

**Partially Verified.**

- «Репо»-числа — измерены непосредственно (Verified-уровень).
- «GSC»-числа — из внутреннего отчёта проекта; сам экспорт GSC в репозитории
  не лежит, поэтому перепроверить его нельзя — принято как единственный
  задокументированный замер (это и есть причина Partially).
- Разделы UNKNOWN — не измерено.

## Аддендум 2026-08-25 — банки Невады (точечная сверка)

Только факты этой даты; остальные таблицы документа — состояние на 2026-08-05
(релизы AZ-банков 2026-08-15…08-19 в них не внесены, см. пробел покрытия в
`26_CHANGELOG.md`).

| Метрика | Значение | Источник |
|---|---|---|
| `nv-cms` | 500 вопросов × EN/RU/ES (1500 позиций), ключи A125 B125 C125 D125 | ветка `content-banks-src` осн. репо, `js/questions/nv-cms*.js` (финальный коммит `ec23475`) |
| `nv-b` | 500 вопросов × EN/RU/ES (1500 позиций), ключи A125 B125 C125 D125 | ветка `content-banks-src`, `js/questions/nv-b*.js` (финальный коммит `aefecbd`) |
| CSV платных банков | 33,500 строк / 22 банка / ~35 MB (gitignored; ждёт импорта в Supabase `bank_questions`) | `scripts/generate-bank-csv.js` (ветка `content-banks-src`, `cb9e368`), вывод генератора 2026-08-25 |
| Платных банков всего (со статикой или на `content-banks-src`) | 22 | PAID-список `scripts/generate-bank-csv.js` |

## Verification Status (аддендум)

Partially Verified — строки аддендума проверены чтением названных файлов и
выводом генератора CSV 2026-08-25; таблицы выше аддендума не перепроверялись
с 2026-08-05.

## Аддендум 2026-09-03 — банки Невады nv-b2 и nv-c2 (точечная сверка)

| Метрика | Значение | Источник |
|---|---|---|
| `nv-b2` | 500 вопросов × EN/RU/ES (1500 позиций), ключи A125 B125 C125 D125 | ветка `content-banks-src`, `js/questions/nv-b2*.js` (финальный коммит `c45e642`; строка `26_CHANGELOG.md` от 2026-08-25) |
| `nv-c2` | 500 вопросов × EN/RU/ES (1500 позиций), 5 блоков × 100, ключи по блокам 25/25/25/25 ⇒ A125 B125 C125 D125; 1,330 якорных проверок по корпусу NEC 2017 / 29 CFR 1926 K,V | ветка `content-banks-src`, `js/questions/nv-c2.js`, `nv-c2.ru.js`, `nv-c2.es.js` (финальный коммит `14d9a15`); `docs/content/nv-c2-ledger.md` (ветка осн. репо, до `cb21705`) |
| CSV платных банков | 36,500 строк / 24 банка / ~38 MB (gitignored; ждёт импорта в Supabase `bank_questions`) | `scripts/generate-bank-csv.js` (ветка `content-banks-src`, `499498a`), вывод генератора 2026-09-03 |
| Платных банков всего (со статикой или на `content-banks-src`) | 24 | PAID-список `scripts/generate-bank-csv.js` |
| Запись bank-updates для NV | 5 из лимита 6 на штат | `js/bank-updates.js` (ветка осн. репо, `787a1e4`) |

## Verification Status (аддендум 2026-09-03)

Partially Verified — строки nv-c2, CSV и bank-updates проверены чтением названных
файлов и выводом генератора CSV 2026-09-03; строка nv-b2 взята из
`26_CHANGELOG.md`/`28_AI_CONTEXT.md` (коммит `c45e642` не перечитывался).

## Аддендум 2026-09-03 (поздний) — nv-c21 (точечная сверка)

| Метрика | Значение | Источник |
|---|---|---|
| `nv-c21` | 500 вопросов × EN/RU/ES (1500 позиций), 5 блоков × 100, ключи по блокам 25/25/25/25 ⇒ A125 B125 C125 D125; 1,195 якорных проверок по корпусу UMC 2018 / 40 CFR 82 F / 29 CFR 1926 J,D,K | ветка `content-banks-src`, `js/questions/nv-c21.js`, `nv-c21.ru.js`, `nv-c21.es.js` (финальный коммит `fbf68f0`); `docs/content/nv-c21-ledger.md` (ветка осн. репо) |
| CSV платных банков | 38,000 строк / 25 банков / ~41 MB (gitignored; ждёт импорта в Supabase `bank_questions`) | `scripts/generate-bank-csv.js` (ветка `content-banks-src`, `06ac307`), вывод генератора 2026-09-03 |
| Платных банков всего | 25 | PAID-список `scripts/generate-bank-csv.js` |
| Записей bank-updates для NV | 6 из лимита 6 на штат | `js/bank-updates.js` (ветка осн. репо, `19dc7eb`) |

## Verification Status (аддендум 2026-09-03, поздний)

Verified — строки проверены чтением названных файлов и выводом генератора CSV
2026-09-03.

## Аддендум 2026-09-12 — контент и страницы после цикла `keen-hamilton`

Только факты этой даты; таблицы выше не перепроверялись.

| Метрика | Значение | Источник |
|---|---|---|
| Платных SKU в продаже | **25** (было 24) — добавлен `la-business-law` | `scripts/check-paid-sync.js` («6 lists, 25 courses») |
| Готовых, не запущенных банков | **0** (было 1) — Луизиана подключена; ждёт только импорта CSV владельцем | `js/catalog/la.js`, `supabase/functions/stripe-checkout/index.ts` |
| Активных штатов | 4 (`ca`, `az`, `nv`, `la`) | `js/catalog.js` |
| CSV банка `la-business-law` | 1500 строк (500 EN + 500 ES + 500 RU), 5 блоков × 100, 31 секция, ключи 125/125/125/125 | генерация 2026-09-12 по файлам ветки `content-banks-src` (в репозиторий не коммитится) |
| Сэмпл-наборов для SEO-страниц | **30** (было 27) | `js/samples/` |
| Practice-страниц | 30 экзаменов × 3 языка; из них платных с проверкой оффера — 72 | `ls practice/`, `scripts/check-offer.js` |
| URL в `sitemap.xml` | **156** (было 147) | `sitemap.xml` |
| Панелей источников `COURSE_REF` | 31 (было 30) | `scripts/check-course-ref.js` |
| Записей `bank-updates` по штатам | ca 6, az 6, nv 6, la 1 (лимит — 6 на штат) | `js/bank-updates.js` |

### Verification Status (аддендум 2026-09-12)

**Verified** — каждое число измерено прогоном соответствующего скрипта или
подсчётом по файлам в момент правки. Данных о продажах и трафике этот
аддендум не содержит: `user_courses`, `course_trials` и
`marketing_page_daily` из репозитория недоступны — UNKNOWN до выгрузки
владельцем.

## Аддендум 2026-09-13 — первые измеренные данные о продажах и трафике

Источник: выгрузки владельца из Supabase SQL Editor (2026-09-12 и 2026-09-13);
у AI-сессии доступа к базе нет, числа приведены дословно со скриншотов.

| Показатель | Значение |
|---|---|
| Аккаунтов всего | **19** (по числу строк авто-выдаваемого `contractor-business`) |
| Платящих подписок (`stripe_subscription_id is not null`) | **6** — по одной на `cslb-law`, `c10-exam`, `b-general-building`, `c46-solar`, `c8-concrete`, `c36-plumbing` |
| Курсы с интересом, но без оплат | `c20-exam` (8 строк, 3 активных, 0 платящих), `c27-landscaping` (5 строк), `epa-608` (10, бесплатный) |
| Платящих в AZ и NV | 0 |
| Триалов всего (`course_trials`) | **34** на 15 курсах; максимум по 5 — `b-general-building`, `c27-landscaping`, `cslb-law` |
| Конверсия триал → оплата | 6 из 34 ≈ 18% (вычислено из двух чисел выше) |
| Просмотры за 30 дней (`marketing_page_daily`) | `/` 361 (engaged 131), `/app.html` 275 (85), `/course.html` 150 (57), `/index.html` 37 |
| Practice-страницы за 30 дней | ≈350 просмотров суммарно; топ: `nicet-fire-alarm` 34, `b-general-building` 30, `epa-608` 23 EN + 22 ES + 9 RU, `az-sre-practice` 23, `c-33-painting` 22 |
| Гайды за 30 дней | в топ-30 только два: `es/guides/c-27-landscaping-license-california` 15, `guides/california-arizona-contractor-reciprocity` 7 |
| Испанские страницы | идут наравне с английскими: `es/practice/epa-608` 22 против 23 EN; `es/practice/cslb-law-and-business` 16 против 12 EN |

Ограничения: окно 30 дней; `visitors` в таблице нет — считаются `views` и
`engaged_visitors`; тестеры на уровне устройств не отфильтрованы; строки
`user_courses`, созданные во время бесплатной беты (до 2026-08-01), не
отличимы здесь от покупательского интереса.

Исправление живых данных 2026-09-13: строка `user_courses` с
`course_id = 'osha-hvac'` (курс заменён на `osha-construction` ещё коммитом
`3d384b4`) переведена владельцем на действующий id.

### Verification Status (аддендум 2026-09-13)

**Partially Verified** — числа приведены дословно из результатов SQL-запросов,
выполненных владельцем (скриншоты); определения метрик проверены по коду
`supabase/sql/marketing-daily-aggregates.sql` и `trial-3day.sql`. Производные
величины (18% конверсии, ≈350 просмотров практики) вычислены из приведённых
чисел и помечены как вычисленные.

## Аддендум 2026-09-16 — внешние ориентиры спроса по классификациям CSLB (для выбора следующего банка)

Источники прочитаны 2026-09-16 (запрос владельца «какой банк добавить»):

- **Действующие лицензии CSLB по классификациям на 2020-12-31** (Wikipedia «California
  Contractors State License Board», со ссылкой на данные CSLB; всего 229 909 активных лицензий
  в 44 классификациях): B 103 223 · C-10 25 875 · C-36 15 859 · C-33 15 766 · **A 14 780** ·
  C-20 12 294 · C-27 11 749 · **C-15 7 222** · C-54 6 580 · C-8 6 329. Более свежих
  посчитанных данных не найдено: страница статистики CSLB не существует (404), Data Portal
  отдаёт только интерактивные списки. Из первой десятки без банка в LICENA — только **A General
  Engineering** и **C-15 Flooring**; остальные восемь уже есть. Классификации ниже десятки
  (C-5, C-9, C-35, C-29, C-13, C-39, C-46 …) — счётов нет (UNKNOWN).
- **Испанские гиды CSLB** (`www2.cslb.ca.gov/Contractors/Applicants/Examination_Study_Guides/`):
  ES-версия есть у ВСЕХ классификаций A, B, B-2, C-2…C-60, ASB, HAZ, LAW; у C-61 гида нет.
  Признак «есть ES-гид» больше не отличает трейды (в `docs/marketing/growth-narrow-first-2026-07.md`
  он использовался как довод за C-27).
- **Доступ к кодам на up.codes (анонимно, 2026-09-16):** CRC 2025 — открыт (200); CBC 2025 —
  главы открыты (подтверждено ранее `docs/content/c54-blueprint.md`); **CMC 2025 — закрыт логином**
  (`missingRequiredLogin: true`), как и NEC/UPC/UMC для Невады.
- Semrush MCP: API-юниты исчерпаны, объёмы поисковых запросов — UNKNOWN.
- Собственные данные LICENA (см. аддендум 2026-09-13 выше): все 6 оплат — Калифорния; AZ и NV —
  0 оплат при 4 + 5 платных банках; practice-страницы за 30 дней: `nicet-fire-alarm` 34,
  `b-general-building` 30, `az-sre` 23, `c-33` 22.

## Аддендум 2026-09-16 — GSC: топ-40 запросов по показам (снимок `query,page`, окно 2026-08-14…2026-09-10)

Источник: SQL-запрос владельца к `public.gsc_snapshots` (последний снимок с
`payload->'dimensions' = ["query","page"]`, скриншот 2026-09-15). Числа — показы
(impressions) / клики за 28 дней; кластеры и суммы вычислены из 40 строк результата.

- **NICET / fire alarm** — ≈183 показа (`nicet certified fire alarm design` 88,
  `nicet fire protection engineering` 62, `nicet certified` 6, `nicet` 5, `nicet certification` 5,
  `fire alarm technician certification` 4, `fire alarm certification` 3, `fire alarm system
  certification` 3, `nicet exam` 2, `nicet verification` 2, `nicet level i` 2, `how to get nicet
  level 1` 1), 0 кликов.
- **HVAC / C-20** — ≈77 показов (`how to get heating and air conditioning license` 42,
  `hvac license california` 7, `c20 contractor` 6, `c20 license` 4, ещё 9 вариантов по 1–3),
  0 кликов.
- **C-33 painting** — 22 показа (`c33 license` 10, `c-33 license` 7, `california painting
  contractor license test` 3, `c33 practice test free` 2), 0 кликов.
- **Arizona** — 14 показов (`arizona business management exam` 6, четыре запроса о reciprocity
  CA↔AZ по 2), 0 кликов.
- **Бренд** — `licena` 19 показов, 2 клика. Прочее: `license b california` 3,
  `how to get a c-8 classification` 2, `landscape licence california` 1,
  `how many tries do you get` 1 показ / 1 клик.
- **Всего кликов в топ-40 — 3** (2 брендовых + 1). Показы есть только у тем, для которых на сайте
  уже есть страницы (`practice/nicet-fire-alarm`, `practice/c-20-hvac`, `practice/c-33-painting`,
  `roadmap-az.html`, `guides/arizona-contractor-license`). Запросов про A General Engineering и
  C-15 Flooring в топ-40 нет — страниц по этим классификациям на сайте нет, поэтому Google их
  не показывает; спрос по ним из GSC оценить нельзя (UNKNOWN).

### Verification Status (аддендум 2026-09-16, GSC)

**Partially Verified** — строки взяты дословно со скриншота результата SQL; суммы по кластерам
вычислены; позиции (position) в запрос не входили — UNKNOWN.

## Аддендум 2026-09-16 (поздний) — банк A `a-general-engineering` (точечная сверка)

| Метрика | Значение | Источник |
|---|---|---|
| `a-general-engineering` | 500 вопросов × EN/RU/ES (1500 позиций), 5 блоков × 100, ключи по блокам B1 A24 B27 C24 D25, B2–B5 25/25/25/25 ⇒ A124 B127 C124 D125; числовой паритет вариантов и условий RU/ES к EN — 0 расхождений на 1–500 | ветка `content-banks-src`, `js/questions/a-general-engineering.js`, `.ru.js`, `.es.js` (финальный коммит `0fa77f5`); `docs/content/a-general-engineering-ledger.md` (ветка осн. репо, до `9275489`) |
| CSV платных банков | 39,500 строк / 26 банков / ~44 MB; отдельный файл банка A — 1,500 строк / ~2.3 MB (оба gitignored; ждут импорта в Supabase `bank_questions`) | `scripts/generate-bank-csv.js` (ветка `content-banks-src`, `ea72076`), вывод генератора 2026-09-16 |
| Платных банков всего | 26 | PAID-список `scripts/generate-bank-csv.js`; `scripts/check-paid-sync.js` (ветка осн. репо `bb4d74c`: 6 списков, 26 курсов) |
| Записей bank-updates для CA | 6 из лимита 6 на штат (запись 2026-07-21 C-27 снята) | `js/bank-updates.js` (ветка осн. репо, `bb4d74c`) |
| Рендер-проверка wiring | 45/45 проверок (кабинет CA, плеер, EN/RU, 1280/360 px, 0 ошибок консоли) | Playwright-сьют и скриншоты в scratchpad сессии, в репозитории отсутствуют |

## Verification Status (аддендум 2026-09-16, поздний)

Verified — строки проверены чтением названных файлов, выводом `scripts/check-banks.js`,
скриптом числового паритета и выводом генератора CSV 2026-09-16.

## Аддендум 2026-09-17 — банк `la-building` (Louisiana Building Construction, точечная сверка)

| Метрика | Значение | Источник |
|---|---|---|
| `la-building` | 500 вопросов × EN/RU/ES (1500 позиций), 5 блоков × 100, ключи 25/25/25/25 в каждом блоке ⇒ A125 B125 C125 D125; цифровой паритет RU/ES к EN по q/opts/re 1–500 — 0 расхождений; 44 арифметических ключа пересчитаны двумя переводчиками — совпадают | ветка `content-banks-src`, `js/questions/la-building.js`, `.ru.js`, `.es.js` (`477ba36`); вывод `scripts/check-banks.js` 2026-09-17 (0 ошибок, 0 предупреждений по банку) |
| Секции по блокам | B1 Sitework & Safety (6 секций); B2 Concrete; B3 Masonry & Metals; B4 Carpentry & Envelope (Wood Framing 45, Exterior Walls 22, Roofing 22, Doors/Windows 11); B5 Occupancy/Types/Egress 42, Plan Reading 20, Quantity Takeoff 27, Bid & Job Arithmetic 11 | `docs/content/la-building-ledger.md` (ветка осн. репо) |
| Финальная сверка ключей | 345 числовых ключей — механическая сверка с сохранённым текстом источников, 31 остаток разобран вручную (все подтверждены); выборка 50 нечисловых (seed 20260917) — 50/50; кросс-блочные дубли — 0 | скрипты `rekey.py`, выборка и Jaccard в scratchpad сессии (в репозитории отсутствуют) |
| CSV банка | 1,500 строк, ~2.0 MB (gitignored) — передан владельцу, ждёт импорта в Supabase `bank_questions` | `scripts/generate-bank-csv.js la-building` (ветка `content-banks-src`, `b029874`), вывод генератора 2026-09-17 |
| Платных банков всего | 27 | `scripts/check-paid-sync.js` (ветка осн. репо `357b526`: 6 списков, 27 курсов) |
| Записей bank-updates для LA | 2 из лимита 6 на штат | `js/bank-updates.js` (ветка осн. репо, `fe4d7f0`) |
| Рендер-проверка wiring | 45/45 (кабинет LA, плеер, EN/RU, 1280/360 px, 0 ошибок консоли) + проход 500 вопросов в RU 360 px без сбоев | Playwright-сьют и скриншоты в scratchpad сессии, в репозитории отсутствуют |

## Verification Status (аддендум 2026-09-17)

Verified — строки проверены чтением названных файлов, выводом `scripts/check-banks.js`,
скриптом цифрового паритета и выводом генератора CSV 2026-09-17.
