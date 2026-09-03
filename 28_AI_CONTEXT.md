# 28 — AI Context: сводный контекст для AI-сессий

Последняя сверка: 2026-08-05 (полная) · 2026-08-11 (точечная: редизайн лендинга) · 2026-08-24 (точечная: Невада) · 2026-08-25 (точечная: мерж NV; nv-cms, nv-b и nv-b2 готовы полностью) · 2026-08-30 (точечная: маркетинговые агрегаты) · 2026-09-03 (точечная: старт nv-c2, статус блокера up.codes; позже — блокер снят, Блоки 2–5 написаны; итог дня — nv-c2 готов полностью, wiring, CSV 36,500/24; вечер — старт nv-c21 Run 1; ночь — Блок 2 nv-c21 написан)
Назначение файла: быстрый ввод в курс дела для любой новой AI-сессии.
Всё ниже — факты из `lalianamen/llicena` на дату сверки; оценок нет.

## Состояние проекта одним абзацем

LICENA (licena.us) — трёхъязычный (EN/ES/RU) тренажёр лицензионных экзаменов
США: статический сайт на GitHub Pages (ветка `main`), бэкенд Supabase
(Auth + Postgres/RLS + Edge Functions), платежи Stripe. Бесплатная бета
закончилась 2026-07-31; сейчас — подписка $20/мес за курс с разовым 3-дневным
триалом. Запущена одна витринная категория — Construction (Калифорния +
Аризона, 14 платных курсов и 7 бесплатных); банки Transportation и Beauty
написаны, но скрыты фильтром `LAUNCH_CATEGORIES` (`js/app-cabinet.js:72`).
Последний запуск — Arizona SRE (2026-08-04, первый курс вне Калифорнии).
Дополнение 2026-08-25: Невада (третий штат кабинета) ВЛИТА в `main`
владельцем (HEAD `961123d`) и живёт на licena.us — NV в `STATES` +
`js/catalog/nv.js` с 4 федеральными бесплатными банками (те же course id,
что в CA/AZ); анализ выполнимости — `docs/content/nv-feasibility.md`
(линейка nv-cms → nv-c2 → nv-c21 → nv-c1d → nv-b по бюллетеням PSI от
25.03.2026). Генерация платных NV-банков ИДЁТ (старт 2026-08-25): `nv-cms` —
блюпринт и fact ledger в `docs/content/nv-cms-blueprint.md`/`nv-cms-ledger.md`
(ветка осн. репо); **банк ГОТОВ полностью** (Run 6, 2026-08-25): на
`content-banks-src` — 500 вопросов × EN/RU/ES (ключи A125 B125 C125 D125,
финальный коммит `ec23475`; 10% выборка ключей перепроверена по
первоисточникам — 50/50), CSV для импорта сгенерирован
(`supabase/sql/bank_questions.csv`, gitignored); wiring на
ветке осн. репо (`beda7fa`): EXAM_FORMATS `"nv-cms" {count:60, passPct:75}`
+ EXAM_MINUTES 120, карточка курса в `js/catalog/nv.js` (подгруппа
`nv-nscb`), COURSE_REF-панель — ВЛИТО в `main` владельцем 2026-08-25
(fast-forward до `034521e`, задеплоено). Остался один шаг до пользователей:
владелец импортирует CSV в Supabase `bank_questions` (до импорта карточка
видна, банк пуст). Порядок линейки изменён 2026-08-25: `nv-c2`/`nv-c21`/`nv-c1d` заблокированы доступом к NEC 2017/UMC 2018/UPC 2018 (up.codes гейтит NFPA/IAPMO-титулы логином; детали — `docs/content/nv-b-ledger.md`, «Source-access verification»). **`nv-b` (B General Building) ГОТОВ полностью** (Runs 1–7, 2026-08-25): на `content-banks-src` — 500 вопросов × EN/RU/ES (ключи A125 B125 C125 D125, финальный коммит `aefecbd`; каждое IBC-число утверждалось дословно в двух адопциях NV+WY; 10% выборка ключей — 50/50; полнобанковый числовой паритет 1–500 по всем полям и языкам; блюпринт `93548c7`, леджер до `790e486`); wiring на ветке осн. репо `claude/question-bank-generation-analysis-y47sk7` (`77d909b`): EXAM_FORMATS `"nv-b" {count:80, passPct:70}` + EXAM_MINUTES 180, карточка в `nv-nscb`, COURSE_REF, запись bank-updates — ЖДЁТ мержа владельцем; CSV регенерирован с nv-b: 33,500 строк / 22 платных банка (~35 MB) — ждёт импорта в Supabase. Дальше линейка СТОИТ до получения NEC 2017 (+NFPA 72), UMC 2018, UPC 2018 или логина up.codes. **`nv-b2` (B-2 Residential & Small Commercial) ГОТОВ полностью** (Runs 1–7, 2026-08-25, по команде владельца «начинай следующий платный банк»): на `content-banks-src` — 500 вопросов × EN/RU/ES (ключи A125 B125 C125 D125, финальный коммит `c45e642`; каждое IRC-число утверждалось дословно в двух адопциях NV+ID — гейт поймал две реальные айдахские поправки, R403.1.1 (резерв NV+OK) и таблицу R302.6; 10% выборка ключей seed 20260825 — 50/50; RU-переводчик независимо пересчитал все 45 арифметических ключей Блока 5 — верны; полнобанковый паритет 3000/3000 полей на язык; blueprint `18d38ef`, леджер до `46a10d2`). Wiring на ветке осн. репо (`ea312ac`): EXAM_FORMATS `"nv-b2" {count:80, passPct:70}` + EXAM_MINUTES 180, карточка в `nv-nscb` после nv-b, COURSE_REF, запись bank-updates (NV 4/6) — ЖДЁТ мержа владельцем; платные списки (6 синхронизированных мест) не тронуты по прецеденту nv-b — их включает владелец со Stripe-продуктом. CSV регенерирован с nv-b2: **35,000 строк / 23 платных банка** (~36 MB, gitignored) — ждёт импорта в Supabase. ВАЖНО: доступ up.codes ужался — гл. 1 IRC логин-гейтед для NV и ID, гл. 3 Оклахомы тоже (сохранённые сплющивания гл. 3–10 NV+ID лежат в scratchpad сессии и на диске не в репо); линейка nv-c2/c21/c1d по-прежнему заблокирована (NEC 2017 + NFPA 72 / UMC 2018 / UPC 2018) и при разблокировке имеет приоритет; аккаунт up.codes снял бы блок и для будущих банков. Примечание к цифрам
абзаца: счёт курсов «14 платных / 7 бесплатных» — на дату сверки 2026-08-05;
релизы аризонских trade-банков 2026-08-15…08-19 в этот документ ещё не
внесены (см. пробел покрытия в `26_CHANGELOG.md`). Practice-страницы четырёх AZ trade-банков (`435946f`: 12 страниц +
`js/samples/az-*.js` + 4 slug в `EXAMS` + sitemap 137 URL) ВЛИТЫ в `main`
тем же fast-forward `034521e` (2026-08-25) и живут на проде.
Дополнение 2026-08-28: запущен License Roadmap Phase 1 (`0d7fcfb` осн.
репо, влито в `main`) — начало расширения из тренажёра в платформу
сопровождения лицензирования: `/roadmap.html` (анкета 8 экранов →
персональный чек-лист 11 шагов CSLB, EN/ES/RU, прогресс по весам шагов),
конфиг-архитектура под новые штаты (`js/roadmap/roadmap-config.js`),
персистентность Supabase (`supabase/sql/license-roadmaps.sql` — применяет
владелец; статус применения UNKNOWN) + localStorage-фолбэк, входы с
лендинга и из кабинета, превью `/roadmap.html?demo=1`. Детали —
`13_UX.md`, схема — `04_DATABASE.md`. Post-license фазы (бизнес-сетап,
маркетинг и т. п.) сознательно НЕ строились (границы фазы 1).
Вторая итерация 2026-08-28 (`2c71fc6` осн. репо, влито в `main`) — логика
персонализации: derive() завершает classification/experience только при
принятии заявления CSLB (не при подаче), новый движок recommendNext()
(«Your next step» по каскаду приоритетов + CTA), блок «While you wait»,
маркер «You are here» по user-reported этапу (`ROADMAP_STAGE_POSITION`),
подпись «LICENA recommendation — not a CSLB determination», контекстные
подписи вех mc_*; схема БД и события Clarity не менялись; e2e — 95
проверок с персонами A–J. Детали — `13_UX.md`.
Дополнение 2026-08-29: Application Assistant Phase 1 (`2c66f13` осн. репо,
влито в `main`) — `/application.html`: ведомая подготовка официального
заявления CSLB (CA · Original · Sole Owner) с картой 1:1 на формы 13A-1
(rev. 01/2026) и 13A-11, FieldExplanation на сложных полях, опытным
ассистентом, 3-уровневой валидацией, печатным Review Packet и подачей
только через официальные каналы CSLB (портал SimpliGov для sole owner /
Easy-Fill / почта; API третьих сторон у CSLB нет; статус «подано» — только
со слов пользователя). Файлы: `js/roadmap/application-config.js`,
`i18n-application.js`, `app-application.js`, `css/application.css`,
`supabase/sql/license-applications.sql` (владелец ещё должен применить;
статус UNKNOWN). SSN/ITIN, дата рождения, водительское удостоверение НЕ
собираются. Интеграция в Roadmap: CTA шага 3 + user-reported подача.
Детали — `13_UX.md`, схема — `04_DATABASE.md`.
Дополнение 2026-08-29 (2): Adaptive Licensing Intake v2 (`037bdd9` осн.
репо, влито в `main`) — анкета роадмапа переписана на условные ветвящиеся
экраны по реальному пути CSLB (entity/DBA, заявление со статусами и датами,
опыт с удостоверяющим, Live Scan, экзамены Law и trade раздельно, финальные
требования, выдача, напоминания); «не уверен в классификации» удалён;
дашборд «за секунды» + чек-лист «Before you start working» после выдачи
(условный, официальные источники, нейтральный финал без Phase 2); v1-модели
мигрируют на лету (`migrate()`, обратимо). Новый SQL
`supabase/sql/license-roadmaps-v2.sql` (12 колонок дат/entity/reminder;
применяет владелец; статус UNKNOWN; клиент до применения падает обратно на
v1-строку). ВАЖНО: reminder_opt_in хранится, но инфраструктуры доставки
писем/push в LICENA НЕТ — подсказки только на дашборде. Детали —
`13_UX.md`, схема — `04_DATABASE.md`.
Дополнение 2026-08-27: у трёх невадских платных банков появились
practice-страницы (`a25d54e` осн. репо, влито в `main`): слаги
`nv-cms-exam` / `nv-b-general-building` / `nv-b2-residential` × EN/ES/RU
(9 страниц по шаблону AZ), сэмплы `js/samples/nv-{cms,b,b2}.js`
(8 оригинальных вопросов на банк, ключи пересверены по первоисточникам),
`js/seo.js` EXAMS +3, `sitemap.xml` 137→146 URL. Детали — `12_SEO.md`.
Дополнение 2026-08-26: подключён Google Analytics 4 (`fca5fb2` осн. репо,
влито в `main`): Google tag `G-1YE5GDRVFZ` на всех 171 обслуживаемых
страницах — загрузчик gtag.js в `<head>` + конфиг во внешнем `js/ga.js?v=1`
(CSP без `'unsafe-inline'`), CSP 167 страниц расширен доменами
googletagmanager/google-analytics/analytics.google.com. Детали и покрытие —
`14_ANALYTICS.md`; `privacy.html` про cookies `_ga` пока не обновлялся
(за владельцем).

Дополнение 2026-09-03: начат `nv-c2` (C-2 Electrical Contractor) — Run 1
заблокированной линейки NV trade-банков (nv-c2 → nv-c21 → nv-c1d), по
решению владельца достроить Неваду прежде открытия нового штата. На ветке
осн. репо `claude/bank-questions-state-choice-vafhir` (`7e939ba`, в `main`
НЕ мержилось): `docs/content/nv-c2-blueprint.md` (outline 15 областей = 80
вопросов дословно из PSI CIB «C» upd. 3/25/2026, перефетчен 2026-09-03 —
ревизия не менялась; формат 80/56/4 ч open book; блоки 5×100; дедуп-регистр
против nv-cms/nv-b/nv-b2/nicet-fire-alarm) + `docs/content/nv-c2-ledger.md`
(скелет, source-access verification). Статус блокера ИЗМЕНИЛСЯ: главы
NEC 2017/UMC 2018 на up.codes по-прежнему login-gated, но гейт теперь
предлагает «free … no credit card required» ⇒ разблокировка = бесплатный
аккаунт up.codes (действие владельца). Блок B1 (100: доктрина/арифметика +
29 CFR 1926 Subparts K/V, открытые источники) авторизуем ДО разблокировки;
B2–B5 (400, NEC-сорсинг) — только после. Из CIB: PSI предлагает C-2 на
испанском («NV C-2 Electrical - Spanish») — заметка для будущей
practice-страницы.

Дополнение 2026-09-03 (позже в тот же день): блокер `nv-c2` СНЯТ и Блоки 2–5
НАПИСАНЫ. Владелец создал бесплатный аккаунт up.codes и экспортировал
залогиненные страницы Nevada NEC 2017 (гл. 1–9) в `.doc`/`.md`; заголовок
страницы «Adopts Without Amendments — NFPA 70, 2017» ⇒ одной адопции
достаточно (леджер, «Source acquisition — NEC 2017»). 29 CFR 1926 Subparts
K/V взяты с eCFR versioner API. Тексты кодов — ТОЛЬКО в scratchpad сессии
(вне репо; при новой сессии владелец загружает экспорты заново). На
`content-banks-src`: `js/questions/nv-c2.js` = 400 вопросов EN, ids
101–500 (Блоки 2–5, ключи 25/25/25/25 в каждом блоке, чекер 0 ошибок;
последний коммит `cecb762`); леджер `docs/content/nv-c2-ledger.md` на ветке
`claude/bank-questions-state-choice-vafhir` до `1f64134` (по строке на id:
факт → NEC 2017 § → ключ; кросс-блочные «do not re-key» заметки). Блок 1
(ids 1–100: NEC Art. 100/110 + арифметика 30, 29 CFR 1926 Subpart K 45,
Subpart V 25) — Run 6 запущен 2026-09-03. Дальше: RU/ES-оверлеи, финальная
проверка, wiring (EXAM_FORMATS nv-c2 80/70 %, EXAM_MINUTES 240, карточка
`nv-nscb`, COURSE_REF, bank-updates NV — сейчас 4/6), PAID-список CSV +
регенерация, затем `nv-c21` (UMC 2018 гл. 1–17 + прил. A–E уже
экспортированы владельцем, тоже без поправок штата) и `nv-c1d` (UPC 2018 —
брать страницу Texas на up.codes: Nevada UPC 2018 с поправками; UMC 2012
Nevada без поправок).

Дополнение 2026-09-03 (итог дня): **`nv-c2` (C-2 Electrical Contractor) ГОТОВ
полностью** (Runs 1–8, 2026-09-03). На `content-banks-src`: `js/questions/nv-c2.js`
+ `nv-c2.ru.js` + `nv-c2.es.js` — 500 вопросов × EN/RU/ES (ключи A125 B125
C125 D125; каждое число NEC утверждалось дословно по тексту 2017 в редакции
Невады без поправок — 1,330 якорей; 10 % выборка ключей 50/50; паритет чисел
оверлеев 0; финальный коммит `14d9a15`; nv-c2 в PAID-списке CSV-генератора
`499498a`). Леджер `docs/content/nv-c2-ledger.md` на ветке осн. репо
`claude/bank-questions-state-choice-vafhir` до `cb21705`. Wiring на той же
ветке (`209c21a`, `787a1e4`): `EXAM_FORMATS "nv-c2" {count:80, passPct:70}`,
`EXAM_MINUTES` 240, карточка в `nv-nscb` после nv-b2, `COURSE_REF` (5
источников), запись bank-updates «NV C-2» (NV 5/6) — ЖДЁТ мержа владельцем.
CSV регенерирован: **36,500 строк / 24 платных банка** (~38 MB, gitignored) —
ждёт импорта в Supabase. Платные списки (6 мест) не тронуты по прецеденту
nv-b — включает владелец со Stripe-продуктом. Practice-страница для C-2 не
создавалась (лейн marketing/design). Тексты NEC 2017 (гл. 1–9) и UMC 2018
(гл. 1–17 + прил. A–E), экспортированные владельцем из up.codes, лежат только
в scratchpad сессии — при новой сессии владелец загружает экспорты заново.
Следующий банк по решению владельца — `nv-c21` (C-21 Refrigeration & Air
Conditioning: PSI 85 вопросов / 60 / 3 ч, UMC 2018 + NFPA 54-2012); затем
`nv-c1d` (UPC 2018 — брать страницу Texas на up.codes, у Nevada поправки;
UMC 2012 Nevada без поправок).

Дополнение 2026-09-03 (вечер): начат `nv-c21` (C-21 Refrigeration and Air
Conditioning Contractor) — Run 1, docs-only, по указанию владельца сразу после
nv-c2: `docs/content/nv-c21-blueprint.md` + `nv-c21-ledger.md` на ветке осн.
репо `claude/bank-questions-state-choice-vafhir` (`6fa2438`). Формат PSI 85 / 60 / 3 ч,
UMC 2018 + NFPA 54-2012; outline 22 темы → 5×100 (Solar right-sized до 4: UMC
гл. 15 — отсылка к проприетарному USEHC; Load Calculations 20 — арифметика с
пересчётом ключей). Корпус UMC 2018 (гл. 1–17 + прил. A–E, экспорт владельца,
без поправок штата) уже в scratchpad сессии; NFPA 54 без открытого текста —
fuel-gas cell только по UMC гл. 13. Дедуп: nv-c2 (NEC), бесплатные `epa-608`
и `osha-construction`, NV-банки. Генерация — с Run 2 (Блок 2 fuel gas).

Дополнение 2026-09-03 (ночь): `nv-c21` Блок 2 (ids 101–200, fuel gas /
combustion air / venting) написан и запушен (`content-banks-src`
`js/questions/nv-c21.js` до `a49cde5`; леджер до `f413e72`), ключи 25/25/25/25,
чекер 0 ошибок. Инцидент API 529 (четыре неудачных запуска, в т. ч. один на
Opus) записан в леджере как «Model note»; итоговый блок написан на основной
модели. Дальше по очереди: Блоки 3, 4, 5, 1 → RU/ES-оверлеи → финальная
проверка → wiring (EXAM_FORMATS 85/70 %, 180 мин, карточка `nv-nscb` после
nv-c2, COURSE_REF, bank-updates — NV станет 6/6, лимит) → CSV.

## Ключевые инварианты (нарушение = сломанный прод)

1. **Деплой = push в `main`**: GitHub Pages отдаёт ветку как есть. Никаких
   секретов и платных банков в `main`.
2. **Платные банки не в `main`**: исходники — только ветка `content-banks-src`;
   сайт читает их из Supabase `public.bank_questions` (RLS). Правка платного
   банка: checkout ветки → правка + чекер там → `node
   scripts/generate-bank-csv.js` → владелец реимпортирует CSV в Supabase.
3. **Формы банков**: платные Construction — всегда 5 блоков × 100 (500);
   бесплатные сертификационные right-sized: `epa-608` 4×50, `osha-construction`
   5×50, `asbestos` 5×20 (owner decision 2026-07-16); остальные вертикали —
   50 вопросов / 6 секций.
4. **Переводы банков**: RU/ES — оверлеи по индексу; `opts` никогда не
   переупорядочивать (`correct` живёт только в EN-файле — реордер молча ломает
   ключи, рендер-проверка этого не увидит). EN+RU+ES правятся одним коммитом.
5. **Хуки DOM**: никогда не переименовывать/удалять `id`/`class`/`data-*`,
   которые читает JS другого файла (лейны редактируют разные файлы).
6. **После любой правки `js/questions/*`** — запускать `node
   scripts/verify.js`. Рендер-проверка обязательна: скриншоты desktop + 360px,
   EN + RU, клик по одному потоку; вопрос в плеере искать по `id`, не по
   номеру на экране (порядок перемешивается).
7. **Журнал `js/bank-updates.js`**: каждое изменение банка — честная запись
   (дата, банк, что сделано, источники, год данных); максимум 6 записей,
   новая вытесняет старейшую.
8. **Контент — только открытые официальные первоисточники**, формулировки
   с нуля; из реальных экзаменов и коммерческих банков не копировать.
   Генерация банков — строго по SOP `docs/content/bank-playbook.md`.
9. **Пять агентных лейнов** (design / content / resources / marketing / smm) —
   каждый правит только свои файлы; marketing не трогает чужие файлы, а выдаёт
   спеки; smm никогда не постит сам (правила — `CLAUDE.md` основного репо и
   `.claude/agents/*.md`).

## Где что лежит (карта навигации)

- Каталог курсов: `js/catalog.js` + `js/catalog/ca.js` / `az.js`
  (штаты TX/FL/NY заведены, `active:false`).
- Список платных курсов держится синхронно в **трёх** местах: `PAID` в
  `supabase/functions/stripe-checkout/index.ts`, `PAID_SUBS` в
  `js/app-cabinet.js`, `PAID_COURSES` в `js/app-course.js` (+ список в
  `scripts/generate-bank-csv.js`).
- Цена: `PRICE_CENTS = 2000` ($20/мес, owner decision 2026-07-14) — в
  `stripe-checkout`. Доступ выдаёт только `stripe-webhook` → `user_courses`.
- Триал: `supabase/sql/trial-3day.sql` (owner decision 2026-08-01), RPC
  `start_trial`, таблица `course_trials` (никогда не чистится — щит от
  фарминга).
- Анти-шеринг: `supabase/devices_anti_sharing.sql` — `register_device()`,
  5 устройств, swap ≤ 1 раза в 30 дней; клиент `js/devices.js` fail-open;
  лимит продублирован константой `MAX_DEVICES` в `js/i18n-app.js` (копия и код
  обязаны совпадать).
- SEO-голова: только `js/seo.js` (лейн marketing) + `sitemap.xml` + `robots.txt`;
  при правках HTML-страниц бампать `?v=` по спеке marketing.
- AI-саппорт: `supabase/functions/assistant/index.ts`, модель
  `claude-sonnet-4-6`; список фактов о продукте зашит в промпт функции —
  при изменении продукта его тоже надо обновлять.
- Автопостинг соцсетей: положить `.txt` в `docs/smm/queue/` (Telegram) или
  `docs/smm/queue-fb/` (Facebook) и запушить в `main` — workflow постит сам.
- Контент-аудит банков: очередь `docs/content-audit/queue.json`
  (статусы pending/done, политика «UNVERIFIED флагуется, не правится»).
- Детальные разборы в базе знаний: функции — `06_FUNCTIONS.md`; модель
  доступа — `07_AUTH_ACCESS.md`; безопасность (в т.ч. статусы
  security-todo) — `08_SECURITY.md`; платежи — `09_PAYMENTS.md`;
  контент-модель и SOP банков — `10_CONTENT_MODEL.md`; локализация —
  `11_I18N.md`; SEO — `12_SEO.md`; аналитика — `14_ANALYTICS.md`; реальные
  числа (включая первый GSC-экспорт 2026-07-17→08-02: 156 показов,
  4 клика) — `15_METRICS.md`; производительность — `16_PERFORMANCE.md`;
  техдолг — `17_TECH_DEBT.md`; зависимости — `18_DEPENDENCIES.md`;
  инфраструктура — `19_INFRASTRUCTURE.md`; проверки — `20_VERIFICATION.md`;
  агентный процесс — `21_AGENT_PROCESS.md`.

## Незапущенное / в работе (на 2026-08-05)

- Банк `la-business-law`: 500 EN + RU/ES-оверлеи готовы на `content-banks-src`
  (последние коммиты ветки), в каталоге и Stripe-карте отсутствует — не
  запущен. Что за экзамен и дата запуска — UNKNOWN.
- Армянский язык (`hy`): staged на курсе C-20 (`testLangs:["hy"]` в
  `js/catalog/ca.js` + `c20-exam.hy.js` на ветке банков) — виден только
  тестерам (`profiles.is_tester`).
- Тестерские триалы продлены до 31 октября
  (`supabase/sql/extend-tester-trials-oct31.sql`).
- (дополнение 2026-08-10, итог `abd8804`) HVAC-калькулятор: страницы
  `/tools/hvac-sizing-calculator/` EN/ES/RU (`js/hvac-calc.js`,
  `css/tools.css`) — COURSE-GATED: открываются только из плеера курса
  (`c20-exam`) через сворачиваемую секцию «Инструменты» в левом сайдбаре
  (свёрнута по умолчанию); клик ставит штамп `lp:tool_gate`, без свежего
  штампа страница редиректит на `/`; `noindex`, в sitemap НЕ входят, ссылок
  с лендинга/гайдов нет. Все секции справочной панели плеера теперь свёрнуты
  по умолчанию. Реестр `COURSE_TOOLS` per-course — калькуляторы добавляются
  любому банку одной записью; с `4234bff`+`6530354` — 10 калькуляторов
  (`TOOL_DEFS`: hvac, hvacdesign (Manual S/D для C-20), electrical,
  plumbing, roofing, concrete, painting, solar, fire, pricing) ТОЛЬКО на
  платных банках (решение владельца 2026-08-11: retention-ценность
  подписки; epa-608 и nicet удалены из реестра), 27 страниц `/tools/**`
  ×3 языка, общая логика `js/trade-calcs.js`. ВАЖНО: страницы `/tools/` не подключают
  `js/seo.js` (он не знает этот путь и перезаписал бы head данными
  лендинга); новые публичные пути либо добавлять в его детекторы, либо
  нести статический head без него.
- (дополнение 2026-08-11, `1e61960` → порт макета `04a2d99`) Лендинг
  `index.html` — точный порт утверждённого владельцем макета-синтеза
  Lovable-образцов (первую «эволюционную» версию владелец отклонил: «сделай
  по макету»). Тема — `css/landing.css` (грузится ТОЛЬКО index.html;
  переопределяет токены после `styles.css`, банды full-bleed через
  margin-breakout + `body{overflow-x:clip}`, auth-модал остаётся светлым);
  интерактив — `js/landing-extras.js` (калькулятор «сколько теряешь без
  лицензии»: ползунки + чипы-пресеты профессий, пресеты — примеры, НЕ
  статистика (BLS за egress-блоком; выдуманные цифры запрещены), живой
  вопрос из `js/samples/c10-exam.js` с двухкнопочным языковым пиллом);
  `js/i18n.js` v45. Секции proof/story и stats-band УДАЛЕНЫ (нет в макете;
  `syncProofShots` защищён if-ами). Hero-фото `img/hero-contractor.jpg`.
  Прежние хуки (`#navLogin`, auth-модал, `data-seo-*`, `testiSection`,
  support) сохранены; `js/seo.js`, sitemap, practice/guides не менялись.
  Переводы вердиктов/CTA, шаблонов окупаемости и единиц ($/час, /мес) JS
  читает из скрытых/встроенных `data-t`-элементов (`#lq-right`,
  `#gc-pb-*`, `.un`) — не переименовывать эти id/классы.

- (дополнение 2026-08-30, уточнено в тот же день) **Маркетинговые агрегаты —
  в `main` и в production.** `main` осн. репо = `ae13124` (мерж выполнен
  2026-08-30 по указанию владельца; ранее в этом же документе значилось, что
  мерж не выполнен — запись отменяется этой). Edge Function
  `marketing-aggregates` задеплоена на живой проект, 4 таблицы `marketing_*`
  созданы, cron `30 15 * * *` активен, записан backfill за 30 завершённых
  PT-дней. Файлы: `supabase/functions/marketing-aggregates/{core.ts,index.ts}`,
  `supabase/sql/marketing-daily-aggregates.sql`,
  `supabase/sql/cron-marketing-aggregates.sql`,
  `scripts/test-marketing-core.mjs`, `scripts/check-channel-parity.js`.
  Ежедневное письмо `daily-stats` — отдельный слой; 2026-08-30 (`c2e9dd6`) в
  нём изменён ТОЛЬКО блок таксономии каналов и подписи строк разбивки.
  Таксономия каналов у двух слоёв обязана оставаться побайтово одинаковой
  (`node scripts/check-channel-parity.js`) — правка бакетов автоматически
  меняет и письмо владельца, и требует переразвёртывания ОБЕИХ функций.
  Бакеты с 2026-08-30: `direct` (пустой `document.referrer`), `flyer`,
  `facebook`, `instagram`, `telegram`, `tiktok`, `search`, `ai`, `other`;
  до этой даты поиск и AI-трафик попадали в `other`. Детали: `06_FUNCTIONS.md`, `04_DATABASE.md`,
  `14_ANALYTICS.md`,
  `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`,
  `tasks/reports/2026-08-30-marketing-analytics-activation-response.md`.

## Расхождения, зафиксированные при сверке (факты, без оценок)

- `README.md` основного репо описывает добета-состояние: «payments (Stripe)
  … still to come», «Free during the beta», упоминает язык `vi` и не упоминает
  каталог `js/catalog/`, Arizona, платные банки. Фактическое состояние — см.
  `01_PROJECT_OVERVIEW.md` и `02_ARCHITECTURE.md`.
- Комментарий в `scripts/generate-bank-csv.js` говорит «nine PAID banks», в
  самом списке `PAID` там 12 позиций, а платных курсов в Stripe — 14.
- `CLAUDE.md` основного репо говорит о «12 paid banks» на `content-banks-src`
  (запись от 2026-07-29); к 2026-08-05 платных курсов уже 14 (+ c33, az-sre).

## Ограничения этой сверки (UNKNOWN)

- Git-история глубже 50 последних коммитов (shallow-клон): дата старта
  проекта, полное число коммитов — UNKNOWN.
- Живые данные Supabase (число пользователей, строки таблиц, состояние
  секретов) — UNKNOWN: в репо только код и SQL, доступа к проду у сессии нет.
- Состояние Stripe-аккаунта (реальные подписки, выручка) — UNKNOWN.
- Данные Google Search Console — первый экспорт (2026-07-17→08-02)
  зафиксирован в `15_METRICS.md` по `docs/marketing/gsc-readout-2026-08.md`;
  более свежие данные GSC — UNKNOWN.

## Правила работы над этой базой знаний

См. `CLAUDE.md` этого репозитория: только реальные данные, `UNKNOWN` для
отсутствующих, обновление документации в том же цикле, что и код; append-only
журналы; `30_CTO_REPORT.md` ведёт ChatGPT — не редактировать; каждый документ
несёт разделы «Source References» и «Verification Status» (стандарт аудита,
`CLAUDE.md` §9).

## Source References

Все пути — в репозитории `lalianamen/llicena`:

- Инварианты и процессы: `CLAUDE.md` основного репо (лейны, формы банков,
  правила переводов, verify-чек-лист, ветка `content-banks-src`),
  `.claude/agents/` (листинг)
- Синхронные списки платных курсов: `supabase/functions/stripe-checkout/index.ts`
  (карта `PAID` + комментарий о `PAID_SUBS`/`PAID_COURSES`),
  `scripts/generate-bank-csv.js` (список `PAID`, строки 10–14)
- Цена/триал/бета: `supabase/functions/stripe-checkout/index.ts`
  (`PRICE_CENTS`), `supabase/sql/trial-3day.sql`,
  `supabase/sql/restore-beta-policies-until-aug1.sql`,
  `supabase/sql/extend-tester-trials-oct31.sql`
- Анти-шеринг: `supabase/devices_anti_sharing.sql` (по ссылке из
  `docs/access-control.md`), `js/devices.js`, `js/i18n-app.js`
  (`MAX_DEVICES`, комментарий о синхронизации с сервером)
- Витрина/каталог: `js/app-cabinet.js:72` (`LAUNCH_CATEGORIES`),
  `js/catalog.js`, `js/catalog/ca.js` (`testLangs:["hy"]`), `js/catalog/az.js`
- Контент: `js/bank-updates.js`, `docs/content/bank-playbook.md` (существование
  и роль — по `CLAUDE.md`), `docs/content-audit/queue.json` (`_about`, `policy`)
- Саппорт: `supabase/functions/assistant/index.ts` (`MODEL`, продуктовые факты
  в промпте)
- Автопостинг: `.github/workflows/facebook-post.yml`, `telegram-post.yml`
  (триггеры по путям `docs/smm/queue*/`)
- Расхождения: `README.md` основного репо, `scripts/generate-bank-csv.js:1`
  («nine»), `CLAUDE.md` основного репо («12 paid banks»)
- Незапущенное: `git log origin/content-banks-src` (la-business-law),
  `git ls-tree origin/content-banks-src js/questions/` (`c20-exam.hy.js`)
- История: git log `main` (50 коммитов shallow-клона)

## Verification Status

**Partially Verified.**

- Проверено чтением названных файлов: все инварианты, списки, константы, пути
  и расхождения.
- Обновление 2026-08-05 (этап 2): `bank-playbook.md`, `check-banks.js`, все 8
  Edge Functions, `docs/access-control.md`, `docs/security-todo.md` прочитаны
  полностью — соответствующие утверждения подтверждены исходниками
  (`06_FUNCTIONS.md`–`10_CONTENT_MODEL.md`).
- Позиции `UNKNOWN` перечислены в разделе «Ограничения этой сверки».
