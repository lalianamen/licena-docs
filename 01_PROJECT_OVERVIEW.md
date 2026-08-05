# 01 — Обзор проекта LICENA

Последняя сверка: 2026-08-05
Источник всех фактов: репозиторий `lalianamen/llicena` (ветки `main` и
`content-banks-src`) в состоянии на дату сверки. Ссылки на файлы даны
относительно корня основного репозитория.

## Что это

**LICENA** — многоязычный тренажёр для подготовки к лицензионным экзаменам США.
Домен: **licena.us** (`CNAME`). Сайт — статический (vanilla HTML/CSS/JS, без
сборки), бэкенд — Supabase, платежи — Stripe (подробно в `02_ARCHITECTURE.md`).

Позиционирование из кода: заголовок лендинга — «LICENA — CSLB & California
Contractor License Practice Tests» (`index.html`); hero-строка — «Pass your CSLB
exam — in your own language» с практикой «for Law & Business, C-10, C-20, C-36
and 14 more California exams», каждый ответ с письменным объяснением на
английском, испанском или русском (`js/i18n.js`).

Дисклеймер (`README.md` основного репо): LICENA — practice tool, не школа; не
аффилирован с CSLB или каким-либо state board; не гарантирует сдачу экзамена.
Вопросы — оригинальные, написаны с нуля по открытым официальным источникам, не
копируются из реальных экзаменов (`CLAUDE.md` основного репо, раздел «Question
authoring»).

`LICENA` — рабочее название («working name — change it freely», `README.md`).

## Языки

- Интерфейс (лендинг, кабинет, плеер): **EN / ES / RU** (`js/i18n.js`,
  `js/i18n-app.js` — по три языковых блока `en/es/ru`).
- Режимы отображения вопросов в плеере: `en | en+ru | ru | es`
  (`js/app-course.js`, шапка файла).
- Армянский (`hy`): существует staged-перевод банка C-20
  (`js/questions/c20-exam.hy.js` на ветке `content-banks-src`; в каталоге курс
  помечен `testLangs:["hy"]` — виден только тестерам, `js/catalog/ca.js`).
- `README.md` основного репо упоминает `vi` (вьетнамский) в описании i18n —
  в актуальном коде языковых блоков `vi` нет (расхождение зафиксировано).

## Штаты и вертикали

Реестр штатов (`js/catalog.js`): **California (active)**, **Arizona (active)**;
Texas, Florida, New York — заведены в `STATES`, но `active:false`.

Категории каталога (`js/catalog/ca.js`, `js/catalog/az.js`):

- **Construction** (CA + AZ) — единственная категория, видимая в кабинете:
  `LAUNCH_CATEGORIES = ["construction"]` (`js/app-cabinet.js:72`).
- **Transportation** (CA: DMV Car/Motorcycle + 8 CDL-курсов) и **Beauty**
  (CA: 6 курсов) — заведены в каталоге, банки написаны, но категории скрыты
  фильтром `LAUNCH_CATEGORIES`.

## Курсы и цены

Модель монетизации (действует с 2026-08-01):

- Бесплатная **бета до 2026-07-31** — курсы самоактивировались без оплаты
  (`supabase/sql/restore-beta-policies-until-aug1.sql`: «free until July 31…
  On Aug 1 re-run stripe-payments.sql SECTION B to lock paid courses again»).
- С запуска платежей: **подписка $20/месяц за курс** (Stripe Checkout,
  `PRICE_CENTS = 2000` — owner decision 2026-07-14,
  `supabase/functions/stripe-checkout/index.ts`), доступ выдаёт только
  `stripe-webhook` после оплаты.
- **Разовый 3-дневный триал** без карты при добавлении платного курса (owner
  decision 2026-08-01, `supabase/sql/trial-3day.sql`); повторный триал на тот же
  курс не выдаётся (таблица `course_trials` не очищается).

**Платные курсы — 14** (карта `PAID` в
`supabase/functions/stripe-checkout/index.ts`): `cslb-law` (Law & Business),
`b-general-building` (B), `b2-residential-remodeling` (B-2), `c7-low-voltage`,
`c8-concrete`, `c10-exam`, `c16-fire-sprinkler`, `c20-exam`, `c27-landscaping`,
`c33-painting`, `c36-plumbing`, `c39-roofing`, `c46-solar`, `az-sre` (Arizona
Statutes & Rules Exam).

**Бесплатные курсы** (лид-магниты, статические банки в `main`;
`js/catalog/ca.js`, подгруппа «Free Courses» + `js/catalog/az.js`):
`osha-construction`, `epa-608`, `asbestos`, `backflow`, `nicet-fire-alarm`,
`nicet-water-based` и гайд `contractor-business` (Business Setup Guide A–Z,
type:"guide", авто-добавляется в «My Courses»). В AZ бесплатный набор — 4
общефедеральных курса (OSHA, EPA 608, два NICET); один id в двух штатах — один
и тот же курс с общим прогрессом (комментарий в `js/catalog/az.js`).

Курсы Transportation и Beauty (16 банков по 50 вопросов) существуют в коде, но
скрыты из кабинета (см. выше); их платность/бесплатность на витрине —
UNKNOWN (категории не запущены, в `PAID` их нет).

## Объём контента (посчитано скриптом по файлам банков, 2026-08-05)

- Ветка **`content-banks-src`** (полный набор исходников): **37 банков,
  10 350 EN-вопросов**; у всех банков есть RU- и ES-оверлеи (для
  `la-business-law` — по коммитам ветки «RU + ES overlays (500 each)»), т.е.
  суммарно ≈ 31 050 позиций на трёх языках.
- Ветка **`main`** (публичные статические банки): **22 банка, 2 850
  EN-вопросов** ×3 языка. Платные банки из `main` удалены 2026-07-29 — статика
  свободно скачивалась; сайт отдаёт их только из Supabase `bank_questions` под
  RLS (`CLAUDE.md` основного репо).
- Форматы банков: платные Construction-банки — **5 блоков × 100 = 500**;
  бесплатные сертификационные — по размеру реального экзамена: `epa-608`
  4×50=200, `osha-construction` 5×50=250, `asbestos` 5×20=100 (owner decision
  2026-07-16); `backflow`, `nicet-fire-alarm`, `nicet-water-based` — 500 (5×100);
  остальные вертикали — 50 вопросов / 6 секций.
- Подготовлен, но **не запущен**: банк `la-business-law` — 500 EN-вопросов
  (5×100) + RU/ES-оверлеи, есть только на `content-banks-src`; в каталоге и в
  Stripe-карте его нет. Что именно это за экзамен и план запуска — UNKNOWN
  (в репо только коммиты ветки).

## Статус и последние вехи (из `js/bank-updates.js` и git-истории `main`)

- 2026-08-04 — запущен **AZ SRE** (первый курс вне Калифорнии, 500×3), плюс
  Arizona-гайды, reciprocity-гайд CA↔AZ, Arizona SMM-пакет (коммиты #182–#187).
- 2026-08-03 — новый платный банк **C-33 Painting** (500×3).
- 2026-07-28 — новый платный банк **B-2 Residential Remodeling** (500×3);
  аудиты платных банков C-8 и C-27 по официальным источникам.
- 2026-08-04 — первый SEO-проход по реальным данным Google Search Console
  (коммит #186; данные — `docs/marketing/gsc-readout-2026-08.md`).

## Детализация в других документах

Архитектура — `02_ARCHITECTURE.md`; структура репозитория —
`03_REPO_STRUCTURE.md`; база данных — `04_DATABASE.md`; API — `05_API.md`;
полный перечень функциональности — `13_UX.md`; хронология —
`26_CHANGELOG.md`.

## Чего в этом обзоре нет (UNKNOWN)

- Дата старта проекта и полная история: локальный клон — shallow (видны
  последние 50 коммитов, 2026-07-28…2026-08-04). Самое раннее датированное
  свидетельство в файлах — `docs/migration-2026-06-22-course-status.sql`.
- Количество пользователей, выручка, трафик — UNKNOWN (живые БД/Stripe вне
  репозитория); измеренные числа проекта, включая первый GSC-экспорт, — в
  `15_METRICS.md`.
- Юридическое лицо / владелец бизнеса — UNKNOWN (в репо не указано; владелец
  GitHub-аккаунта — `lalianamen`).

## Source References

Все пути — в репозитории `lalianamen/llicena`:

- `CNAME`, `index.html` (title), `README.md`, `CLAUDE.md`
- `js/i18n.js`, `js/i18n-app.js` (языковые блоки, hero-строки)
- `js/catalog.js`, `js/catalog/ca.js`, `js/catalog/az.js` (штаты, категории,
  курсы, `testLangs`)
- `js/app-cabinet.js` (`LAUNCH_CATEGORIES`, строка 72)
- `js/questions/*.js` — обе ветки `main` и `content-banks-src`; количества
  вопросов и блоков пересчитаны скриптом (Node `vm`) 2026-08-05
- `js/bank-updates.js` (вехи 2026-07-21…2026-08-04)
- `supabase/functions/stripe-checkout/index.ts` (`PRICE_CENTS`, карта `PAID`)
- `supabase/sql/restore-beta-policies-until-aug1.sql` (граница беты),
  `supabase/sql/trial-3day.sql` (3-дневный триал)
- `docs/migration-2026-06-22-course-status.sql` (самое раннее датированное
  свидетельство), `docs/marketing/gsc-readout-2026-08.md` (существование
  GSC-данных)
- git-история `main` (последние 50 коммитов shallow-клона, 2026-07-28…2026-08-04)
- Коммиты ветки `content-banks-src` (`git log`, верхние: la-business-law)

## Verification Status

**Partially Verified.**

- Проверено чтением источников/пересчётом: объёмы банков (скрипт по файлам обеих
  веток), состав каталога, списки платных/бесплатных курсов, цена и триал,
  языковые блоки, цитаты лендинга, границы беты.
- Взято из самоописаний без независимой перепроверки: содержание записей
  `js/bank-updates.js` (отчёты контент-агента о проверке банков по официальным
  источникам) — сами первоисточники (CSLB, A.R.S. и т.д.) в этой сверке не
  открывались.
- Позиции `UNKNOWN` перечислены в разделе «Чего в этом обзоре нет».
