# Отчёт: маркетинговая подача и конверсия licena.us — 2026-09-20

Статус: **READY_FOR_REVIEW** — изменения на ветке `claude/question-bank-generation-analysis-y47sk7`
осн. репо, коммит `4e2b97d`; в `main` НЕ влито. Превью — раздел 7.
Источник задачи: бриф владельца 2026-09-20 (11 пунктов). Ограничения соблюдены: ИИ-помощник не
добавлялся и не продвигается; цена, длительность пробного периода и правила биллинга не менялись.

## 1. Внесённые изменения (по пунктам брифа)

**П.1 Первый экран.** `index.html` + `js/i18n.js` (EN/ES/RU) + `css/landing.css` + `js/app.js`:
- eyebrow «California · Arizona · Nevada · Louisiana · EN / ES / RU»; бейдж «Free practice questions
  below — no sign-up needed»; подзаголовок: практика конкретного экзамена, разбор каждого ответа,
  три языка, короткие занятия, прогресс сохраняется.
- Главный CTA «Try my exam free — no sign-up →» ведёт в новый блок `#pick`: вкладки штатов
  (CA/AZ/NV/LA; по умолчанию — штат из `lp:state` или Калифорния) и чипы всех 33 экзаменов +
  строка федеральных сертификаций; каждый чип — страница экзамена с бесплатным примером из 8
  вопросов. Второй CTA — роадмап. Пиллы: «15,000+ practice questions», «Every answer explained»,
  «EN / ES / RU». Формулировки для ES/RU написаны отдельно, не калькой.
- Бегущая строка (`.ticker`) заменена пикером.

**П.2 Выбор экзамена и маршрут.** Ссылки пикера и каталога `#exams` при переключении языка
лендинга переписываются на `/es/practice/…` / `/ru/practice/…` (все 33 страницы существуют в трёх
языках). Каталог сгруппирован по штатам; добавлены отсутствовавшие карточки: A General Engineering,
C-54, AZ B/R-11/R-37R/R-39R, NV CMS/B/B-2/C-2/C-21, LA Business & Law / Building Construction.
Роадмап: «California only for now» + фраза «Available for California; other states are not covered
yet» (по `js/roadmap/roadmap-config.js`: `available:true` только у `ca`). Специальность повторно не
запрашивается: с practice-страницы в кабинет уходит `lp:pending_course` (уже было) — теперь ещё и
имя экзамена, и параметр `?next=app.html?subscribe=<course>` в ссылке подтверждения e-mail.

**П.3 Польза до регистрации.** Последовательность вопрос → ответ → разбор → следующий → результат
→ продолжение проверена в Playwright (C-20, 8 вопросов). После ответа: верность, объяснение,
двуязычное объяснение в EN·RU/EN·ES, ссылка «Report an error». Результат — фактический счёт
(ключи из EN-файла), под ним `data-res-note` «Sample score is for practice only…» (есть на всех
33 страницах, проверено `grep`). В FAQ лендинга добавлен вопрос «Does the 8-question sample show
whether I'm ready?» с ответом «No…».

**П.4 Регистрация и возвращение.** `js/sample-quiz.js` v11: переключение режима EN·RU / EN·ES /
RU… пишет `lp:ui_lang` и добавляет `?lang=` ко всем CTA `#authwrap` на странице (включая CTA
экрана результата) → лендинг открывается на этом языке, поле «Preferred language» предвыбирает
его (сценарий из брифа воспроизведён и исправлен). Над полями формы регистрации — строка «You're
continuing: <exam>» (`#rgContinue`, из `lp:pending_course_name`). Ссылка в письме подтверждения
теперь несёт `next=app.html?subscribe=<course>`, поэтому курс открывается даже если письмо
открыто в другом браузере (там `localStorage` пуст). Прогресс: локальные ответы `lp:answers:*`
не трогались; см. расхождение Р-1.

**П.5 Условия покупки.** Сверены с `supabase/functions/stripe-checkout/index.ts` (+ `pricing.ts`:
$19.99 recurring, купон −$10 на первый месяц), `trial-3day.sql` (3 дня без карты, одна выдача на
курс), `js/i18n-app.js` (доступ до конца оплаченного периода). На лендинге: блок цены
«$9.99 for your first month. Then $19.99 a month per exam bank», пункты «Each paid bank is its own
subscription — Law & Business is a separate bank», «Free banks with any account: OSHA, EPA 608,
asbestos, NICET, backflow», сноска «…cancel anytime — access stays until the end of the paid
period»; FAQ «Is Law & Business included with a trade bank? — No…». Бесплатная публичная проба /
бесплатные курсы / пробный доступ к платному банку разведены в тексте `tool4D` и FAQ.
Расхождений с биллингом не найдено (см. также Р-2 про старую формулировку в HTML).

**П.6 Обещания и сравнения.** Удалены: «prepaid package, English-only», «120 hours of video nobody
finishes», «hundreds of $», «A month of LICENA costs less than an hour with a tutor», секции
`#pain`, `#method`, `#cost` и мини-таблица `.offer-table`. Сравнение теперь: школа = полный
предоплаченный пакет (занятия, материалы, расписание) vs LICENA = банк вопросов с разборами,
помесячно, три языка; факт о пересдаче — «CSLB lets you retake after 21 calendar days, PSI charges
the exam fee again» (CSLB Examination FAQ, прочитан 2026-09-17). «Real exam question» → «practice
question»; `tool1D`: «written in exam style from the official CSLB, ROC, NSCB and LSLBC outlines —
not copied from any real exam». Калькулятор дохода: подписи «illustrative estimate, not an earnings
claim» уже были; строка окупаемости переформулирована «In this scenario, the $19.99 monthly
subscription equals …», CTA калькулятора ведёт в пикер. Обещаний сдачи/лицензии/дохода нет.

**П.7 Качество материалов.** C-20 (`practice/c-20-hvac/` EN/ES/RU): фраза «fall protection is
triggered above seven and a half feet, not the federal six» заменена описанием с областью
применения и исключениями — §1670(a) 7½ ft (периметр, незащищённый край, передняя кромка, шахта,
кровля круче 7:12), §1730 (коммерческие пологие кровли: > 20 ft), §1731 (жилые кровли до 7:12:
6 ft и выше; круче 7:12 — любая высота), §1669 (выносные элементы: 15 ft); источники прочитаны на
dir.ca.gov 2026-09-20. В примере `js/samples/c20-exam.js` (вопрос про крышный блок на стройке —
ключ 7½ ft верен по §1670(a)) из объяснения убран выпад «that out-of-state courses teach», добавлено
указание на §1730/§1731. About (EN/ES/RU): фраза «A July 2026 re-check caught… 15 feet to 6; we
updated» не подтверждается репозиторием (в `js/bank-updates.js` и git-истории записи нет) —
заменена на проверяемое описание процесса по `docs/content/bank-playbook.md` (план по официальному
temario, первоисточник и раздел в объяснении, пороги перечитываются при написании, ключи по A–D,
сверка RU/ES с EN по числам, проверка сообщений человеком). Недостающие сведения — раздел 4.

**П.8 Сообщение об ошибке.** Новый `js/report-question.js` (v1): кнопка «Report an error in this
question» → форма (текст + необязательный e-mail). Автоматически прикрепляются course, question_id
(на practice — номер вопроса в примере, в курсе — `q.id`), version (practice: `sample v<N> · quiz
v11`; курс: `player v79`), mode, lang, picked (в курсе — исходная буква и показанная после
перемешивания), url. Запись — ОДНА строка `support_tickets` (kind `complaint`, `course_id`), т.е.
существующий механизм (`js/feedback.js` пишет туда же): триггеры шлют письмо и открывают
GitHub-issue владельцу. Анонимные обращения — с адресом `no-reply@licena.us`; в `ticket-email`
добавлен пропуск письма для него (**нужен редеплой**). После отправки — подтверждение
«Thank you — received. Every report is checked… by a person before anything changes». Банк по
обращению не меняется. Подключено на 99 practice-страницах (`sample-quiz.js?v=11`, старый
Telegram-линк — fallback) и в плеере (`course.html`, `#qReport` под объяснением).

**П.9 Повторы и удобство.** Убраны три дублирующих секции и таблица (см. п.6); «What it is»
собрал четыре аргумента (экзамен · языки/разборы · короткие занятия · попробовать до оплаты).
Дисклеймеры оставлены: hero-fine, блок `.disc`, футер, чекбокс согласия, FAQ «guarantee» —
одинаковых дублей больше нет (FAQ-ответ и блок различаются по содержанию). Топ-бар RU на 1280px
выходил за экран на 55px (CTA «Попробовать бесплатно») — сокращено до «Попробовать».

**П.10 Аналитика.** Существующие события проверены (таблица в `14_ANALYTICS.md`, аддендум
2026-09-20). Добавлены: `sample_completed` (practice, first-party + GA4; раньше был только
Clarity `practice_completed`), `trial_started` (кабинет, first-party + GA4), `purchase_confirmed`
(ТОЛЬКО GA4, срабатывает, когда после возврата со Stripe в кабинете появляется строка
`user_courses` со `stripe_subscription_id`, т.е. вебхук подтвердил оплату), в серверном `purchase`
— `meta.renewal = (billing_reason === "subscription_cycle")` (**редеплой `stripe-webhook`**),
`question_reported`. Дублей покупки нет: в `app_events` покупка — только серверный `purchase`
(его и считает `marketing_weekly_funnel`); клиентский `checkout_completed` — отдельная стадия
«вернулся со Stripe». Персональных данных в событиях нет (course/lang/src/ref/счёт).
Варианты подачи для тестов — раздел 6.

## 2. Замечания, которые уже были исправлены до этой задачи (проверено в коде)
- Калькулятор дохода подписан как иллюстративная оценка, «not an earnings claim» (`gcNote`,
  `gcYearB`, 2026-08).
- FAQ «Are these the real exam questions? — No…» (написаны с нуля по официальным источникам).
- «No pass guarantee» — hero, блок, футер, согласие при регистрации, `honestAck` в кабинете.
- Секция отзывов скрыта и заполняется только одобренными отзывами из БД (`js/reviews.js`);
  выдуманных отзывов на странице нет.
- Канон оффера на всех 81 платной practice-странице (`scripts/check-offer.js`), `data-res-note`
  на всех 33 EN-страницах.
- Ссылки писем подтверждения/сброса с `?lang=` (P8, 2026-09-13); источник привлечения в метаданных
  регистрации (P5); события воронки registration_started / account_created / email_confirmed /
  first_answer / questions_20/100 / checkout_started / checkout_completed / server purchase
  (2026-09-12–13).
- Журнал изменений банков в кабинете (`js/bank-updates.js`).

## 3. Расхождения, требующие решения владельца
- **Р-1 Прогресс.** Ответы курса хранятся в `localStorage` (`lp:answers:<course>`), на сервере —
  только сводка (`sync_progress` → `user_progress`). Тексты «progress is saved» / «your progress is
  never erased» верны в пределах одного устройства и браузера; на новом устройстве ответы не
  восстанавливаются. Решение: серверная синхронизация ответов или смягчить формулировку.
- **Р-2** «5 devices» (`offerL5`, `devReg`) — лимит задан в RPC `register_device` на стороне Supabase;
  в репозитории число не проверяется (UNKNOWN). «Timed mock exam, unlimited attempts» — лимит
  попыток в коде не найден, утверждение не подтверждено и не опровергнуто.
- **Р-3 Мёртвые строки** в `js/i18n.js`: `tcard1Body…tcard4Meta` (выдуманные отзывы с «passed»),
  `story*`, `sc*`, `proof*`, `feat*` — на странице не используются (нет `data-t`), но лежат в
  исходнике. Рекомендация — удалить отдельным коммитом.
- **Р-4 GA4 без e-commerce.** `purchase_confirmed` — кастомное событие без суммы; отчёты дохода GA4
  пусты. Если нужны — Measurement Protocol из `stripe-webhook` (нужен `api_secret` в секретах функции).
- **Р-5 Supabase Redirect URLs.** Ссылка подтверждения теперь `https://licena.us/?lang=<l>&next=…`;
  в Auth → URL Configuration должен быть разрешён шаблон вида `https://licena.us/?*` (или `/**`),
  иначе Supabase подставит Site URL и параметры потеряются (курс тогда откроется только на том же
  устройстве через `lp:pending_course`). Проверить нельзя из репозитория — UNKNOWN.
- **Р-6 About.** «Built by Los Angeles contractors who passed their own exams in a second language»
  оставлено как утверждение владельца; подтверждающих сведений в репозитории нет.
- **Р-7 Клиентский `checkout_completed`** остаётся page-based; для отчётов брать `purchase`
  (сервер). Изменять SQL не требовалось.

## 4. About — чего не хватает (для владельца, на странице не выдумано)
Имена/роли авторов и проверяющих; кто именно разбирает сообщения об ошибках и в какой срок; дата
последней полной перепроверки каждого банка (в журнале есть только изменения); квалификации
(лицензии, номера) — если владелец готов их публиковать.

## 5. Результаты проверок
- `node scripts/verify.js`: course-ref 33 панели, offer 81 страница, `node --check` 147 файлов —
  0 ошибок. `node --check` для всех правленых JS.
- Playwright (сьют `mkt.mjs` вне репозитория, внешние запросы заглушены, Supabase — мок):
  - лендинг EN/ES/RU × 1280/390/360: 4 вкладки пикера, CTA `#pick` / `roadmap.html`, секций
    `#pain/#method/#cost` нет, ссылки каталога и пикера с языковым префиксом, переключение вкладок,
    поле языка формы = язык страницы, горизонтальной прокрутки нет, 0 ошибок консоли (RU 1280 —
    после сокращения CTA).
  - путь проба → регистрация: `/practice/c-20-hvac/` → режим EN·RU → `lp:ui_lang=ru`, все auth-CTA
    получили `lang=ru` → 8 ответов → экран результата → событие `sample_completed` отправлено один
    раз → CTA → `/?src=c20smp&lang=ru#authwrap` → форма на русском, «Вы продолжаете: C-20 HVAC»,
    `lp:pending_course=c20-exam`, имя сохранено.
  - форма ошибки на practice: отправка → POST `support_tickets` с `kind=complaint`, `course_id=
    c20-exam`, сообщение «[Report a question] course=c20-exam · question_id=1 · version=sample v2 ·
    quiz v11 · mode=EN·RU · lang=en · picked=B · url=…», подтверждение показано.
  - плеер (`course.html?id=la-building`, мок банка): до ответа `#qReport` скрыт, после — кнопка;
    e-mail подставлен из сессии; тикет «course=la-building · question_id=8 · version=player v79 ·
    mode=practice · lang=en · picked=D (shown as A)», `user_id` заполнен.
  - превью-сборка (`preview/marketing-2026-09/`, локальная подмена ассетов licena.us): тот же
    переход проба → регистрация работает, 0 ошибок консоли.
- Подтверждение e-mail и повторный вход: логика `emailRedirect` → `nextUrl()` → `app.html?subscribe=`
  прочитана и покрыта существующей проверкой `nextUrl` (только same-site `.html`); живой цикл с
  письмом в песочнице не воспроизводим — UNKNOWN (см. Р-5).
- Скриншоты: `tasks/reports/img/2026-09-20-*.png` (лендинг EN 1280, RU 390, форма ошибки, форма
  регистрации RU, плеер).

## 6. Варианты подачи для последовательных тестов (по одному, с датами)
1. Практика своего экзамена — текущий hero («You know the work. Now get the license.» + пикер).
2. Понятные объяснения — hero-бейдж/пилл: «Every answer explained — see one now», CTA в `#question`.
3. Подготовка на выбранном языке — hero для `?lang=es|ru`: «Экзамен по-английски. Подготовка —
   по-русски» / «El examen en inglés. La preparación, en español». Оценивать по `first_answer` и
   серверному `purchase`, не по кликам; при малом трафике — не менее 4 недель на вариант.

## 7. Превью
- По ветке `main` licena-docs: `https://raw.githack.com/lalianamen/licena-docs/main/preview/marketing-2026-09/index.html`
  (лендинг; `?lang=es` / `?lang=ru` работают) и `…/preview/marketing-2026-09/practice-c-20-hvac.html`
  (страница C-20 с формой ошибки; CTA ведут на превью-лендинг). Неизменённые ассеты грузятся с
  licena.us; регистрация/тикеты с превью пишут в боевую базу — использовать только для просмотра.

## 8. Действия владельца после одобрения
1. Мерж ветки в `main` осн. репо (по команде).
2. Редеплой Edge Functions `ticket-email` (пропуск письма для `no-reply@licena.us`) и
   `stripe-webhook` (`meta.renewal`).
3. Проверить Supabase Auth → Redirect URLs (Р-5).
4. Решения по Р-1…Р-4, Р-6; сведения для About (раздел 4).
