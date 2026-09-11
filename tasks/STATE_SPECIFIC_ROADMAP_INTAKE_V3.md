# TASK — State-Specific Adaptive Roadmap Intake V3

## Status

**RELEASED — California + палитра сайта** (`main` @ `c5c2385`, 2026-09-11); код Arizona и Nevada в `main`, оба `available:false` до решения владельца

2026-09-11, релиз палитры (запись Claude): по команде владельца «так выкладывай в маин»
ветка `claude/state-specific-intake-v3` (`a3656b5`) смержена с `origin/main` (`0b76ec6`,
PR #190/#191; конфликты только в строках `?v=`, правка `.rm-cab` в `cabinet-dark.css`
снята — карточка уже исправлена в `main`), `main` переведён fast-forward на `c5c2385`.
Production: `roadmap-dark.css?v=1`, `roadmap.css?v=20`, `roadmap-config.js?v=15`,
`app-roadmap.js?v=24`; байты ключевых файлов совпадают с репозиторием. Рендер всех
экранов 49/49 после выкладки (первый прогон до fast-forward был пустым из-за упавшего
локального сервера). Arizona/Nevada в `main`, но `available:false`.


2026-09-11, палитра (запись Claude): по запросу владельца «применим цветовую палитру
сайта ко всему родмапу» — коммит `a3656b5` в той же ветке: `css/roadmap-dark.css`
(indigo + gold, как лендинг/кабинет/курс) на четырёх страницах roadmap, блок на лендинге
и карточка кабинета перефиксированы. Рендер всех экранов без ошибок, контраст ≥ 7:1,
превью пересобрано. Мерж/деплой не выполнялись.

2026-09-11, AZ/NV (запись Claude): по запросу владельца «проанализировать аризону и
неваду и на базе проведенного анализа подготовить родмап и туда» в ветке
`claude/state-specific-intake-v3` сделан коммит `eea5c20` поверх `66d1727`. Анализ:
`tasks/reports/2026-09-11-az-nv-roadmap-analysis.md` (ROC — 403, A.A.C. через LII, A.R.S.,
PSI 2477; NSCB, NRS/NAC 624, PSI 270). Аризона: экзамены до заявления (PSI, R4-9-106) —
введён `R.examsFirst`, шаги переставлены, факты экзаменов/сборов/бондов из A.A.C.;
Невада: полный рулсет, новая `roadmap-nv.html`, оверлей `_nv` (171 × 3), кабинет
маршрутизирует по штату. Тестами найдены и исправлены: стирание экзаменов при «заявление
не подано», подпись «10-year window» при 15-летнем окне, семь достижимых CSLB-строк на
AZ/NV. `verify.js` чист, `test-roadmap-v3.mjs` 200, Playwright az-suite2 43 / nv-suite 53,
регрессия CA undo 56 / final 42 из 43 / db-roundtrip 8 / expq 49 из 50 / cv 53 / exp2 68 / biz 41 / dates 34 / bugs 26 из 27 / exp-logic 27; превью пересобрано (AZ/NV в переключателе). Мерж/деплой не
выполнялись. Отчёт: `tasks/reports/2026-09-11-az-nv-roadmap.md`.


2026-09-11, релиз (запись Claude): по команде владельца «ок заливаем родмап пока на
калифорнию» ветка `claude/state-specific-intake-v3` (`4c5d1d5`) смержена с `origin/main`
(конфликты `?v=` в `app.html` / `application.html` / `roadmap.html` решены), `main`
переведён fast-forward на `66d1727`, GitHub Pages отдал сборку через ~15 с (curl:
`app-roadmap.js?v=22`, `roadmap-config.js?v=14`). Возвращены входы на лендинге и в
кабинете, скрытые 2026-09-04. Arizona: `ROADMAP_STATES.az` / `ROADMAP_RULES.az`
`available:false` — `roadmap-az.html` показывает «штат пока недоступен». Проверки на
смерженном дереве: `verify.js`, `test-roadmap-v3.mjs` 183, undo 56, final 42/43 (`8-az`
ожидаемо), db-roundtrip 8, рендер лендинга/кабинета/roadmap/AZ/application. SQL v3 не
применялся — на стороне владельца.


2026-09-11, позже (запись Claude): по спецификации владельца «исправить отмену ручных
отметок выполнения в Roadmap V3» в ветке `claude/state-specific-intake-v3` сделан коммит
`4c5d1d5` поверх `3cb2136`. Баг воспроизведён на `3cb2136`: после «Лицензия выдана» →
«Не начато» `answers.issued` оставался `"y"`, рекомендация и блок «готов к работе»
исходили из выдачи. Исправление: карточки-события (подача, принятие, отпечатки, экзамены,
выдача) при возврате из «Выполнено» открывают подтверждение, затем существующую анкету на
нужном блоке; до подтверждения ничего не сохраняется; после анкеты ответы — единственный
источник; старые противоречивые записи разрешаются вопросом, спорный факт до ответа
маскируется; переставшие применяться ответы откладываются в `answers.aside`; ссылка
«назад к плану без сохранения». Отчёт: `tasks/reports/2026-09-11-roadmap-undo-event-marks.md`
(демо «выдана → исправить → ещё не выдана → перезагрузка», RU 390 px). `verify.js` чист,
`test-roadmap-v3.mjs` 183/183, Playwright undo 56 / final 44 / db-roundtrip 8 / exp-logic 27 /
expq 52 / exp 39 / exp2 68 / cv 53 / cv2 47 / dates 34 / biz 41 / az 53 / bugs 28. Превью
пересобрано (артефакт и `preview/roadmap/licena-roadmap-v3.html`). Мерж/деплой не
выполнялись, миграций нет.


2026-09-11 (запись Claude): по спецификации владельца «завершающие исправления
California License Roadmap V3» в ветке `claude/state-specific-intake-v3` сделаны коммиты
`0cfff69` и `3cb2136` поверх `859fc53`: единая логика следующего шага (карточка, маркер
«Текущий шаг», статусы, кнопка — из `recommendNext()`; «Уточните классификацию лицензии» /
«Уточните сведения об опыте» / подготовка заявления; кнопка открывает вопрос с сохранёнными
ответами); статусы пересчитываются из ответов при каждом показе — «В работе» у подготовки
заявления только при черновике, «Ожидает ответа CSLB» только после подачи, submitted ≠
accepted, будущие этапы «Не начато», ручные отметки помечены и подписаны, флаг сохраняется в
`license_roadmap_steps.metadata` (без миграции); сводка — ячейка «Опыт» только из известных
данных, нейтральная дата, «Завершено X из Y этапов»; карточки — раскрыт рекомендуемый шаг,
выбор в sessionStorage, aria-expanded; подготовка заявления — предупреждение об открытых
вопросах и раздельные кнопки LICENA / сайт CSLB; owner-builder только класс B; Application
Assistant пишет подачу в актуальный ключ плана. Не подтверждено и не менялось: клик по
карточке/ссылке не меняет статус; RU-строка Sole Owner корректна; финальные требования уже
по сущности. Отчёт: `tasks/reports/2026-09-11-roadmap-final-fixes.md` (скриншоты
до/после). `verify.js` чист, `test-roadmap-v3.mjs` 174/174, Playwright final 44 /
db-roundtrip 8 / exp-logic 27 / expq 52 / exp 39 / exp2 68 / cv 53 / cv2 47 / dates 34 /
biz 41 / az 53 / bugs 28. Превью пересобрано по той же ссылке. Мерж/деплой не выполнялись,
миграций нет.


2026-09-10, позже (запись Claude): по спецификации владельца «исправление логики учёта
опыта» в ветке `claude/state-specific-intake-v3` сделан коммит `859fc53` поверх `c17246b`:
работа на себя без лицензии — форма занятости (у CSLB «non-licensed self-employment»),
а не уровень contractor; уровень contractor предлагается только при истории лицензий
(CA или другой штат); для работы на себя без лицензии задаётся вопрос об уровне работы
(да / нет / не уверен); расчёт делит «Заявленный опыт», «Потенциально соответствующий
опыт» и «Требует уточнения» (никогда не ноль при нехватке данных), статус и дисклеймер об
окончательном решении CSLB; предупреждение и кнопка «уточните историю лицензий» удалены;
подтверждающее лицо да / нет / не уверен с понятными статусами и списком ролей CSLB;
документы по таблице CSLB с оговоркой; первый вопрос — «не менее N лет за последние M
лет, который можно подтвердить» (Да / Нет / Не уверен). Источники CSLB прочитаны
2026-09-10. Отчёт: `tasks/reports/2026-09-10-experience-logic.md`. `verify.js` чист,
`test-roadmap-v3.mjs` 174/174, Playwright exp-logic 26 / expq 52 / exp 39 / exp2 68 /
cv 53 / cv2 47 / dates 34 / biz 41 / az 53 / bugs 28. Превью пересобрано с `859fc53`.
Мерж/деплой не выполнялись, миграций нет.

2026-09-10 (запись Claude): по спецификации владельца «упростить блок опыта в
первоначальном опроснике roadmap» в ветке `claude/state-specific-intake-v3` сделан
коммит `c17246b` поверх `ef5fd71`: подробный сбор периодов в первом проходе заменён
одним вопросом «Есть ли у вас около N лет опыта по выбранной специальности, который
вы можете подтвердить?» (три ответа; порог и формулировка — из правил штата,
Arizona — своя); Experience Builder (периоды, уровень/занятость, подтверждающий,
документы, образование, военная служба) открывается из шага опыта плана кнопкой
«Разобрать мой опыт» и правит ответы плана напрямую; у шага опыта — своё
следующее действие по ответу, общий Your Next Action учитывает остальные ответы и
стадию; ни один ответ не даёт «завершено»/прогресс; старые данные не удаляются и
не переинтерпретируются. Попутно исправлено: черновик анкеты при перезагрузке
больше не проходит миграцию v1→v2. Отчёт:
`tasks/reports/2026-09-10-experience-quick-question.md`. `verify.js` чист,
`test-roadmap-v3.mjs` 155/155, Playwright expq 52 / cv 53 / cv2 47 / exp 39 /
exp2 68 / dates 34 / biz 41 / az 53 / bugs 27. Превью пересобрано с `c17246b` по
той же ссылке. Мерж/деплой не выполнялись, миграций нет.

2026-09-05, ещё позже (запись Claude): по спецификации владельца «исправление
противоречий intake и проверка licensing roadmap» (CHANGES_REQUESTED) в ветке
`claude/state-specific-intake-v3` сделан один коммит `ef5fd71` поверх `755f49a`:
исправлены шесть противоречий A–F (уровень «Contractor» без истории лицензий;
устаревший `companyLicensed` после «никогда»; общий номер лицензии; сброс статуса
Law при выборе Trade; reciprocity по факту любой лицензии другого штата; «выдана» =
оба экзамена сданы), правила CSLB сверены с официальными страницами (renew ≤5 лет /
reapply, clear_suspension, условия reciprocity, 18 мес / 21 день / 5 лет, Live Scan
90 дней, сборы, contractor bond, Bond of Qualifying Individual, LLC, workers' comp,
asbestos); holder-вопрос разбит на «на кого лицензия» → «компания уже существует?».
Отчёт: `tasks/reports/2026-09-05-intake-contradictions.md`. `test-roadmap-v3.mjs`
155/155, `verify.js` чист, Playwright: новая `bugs-suite` 27 + регрессия cv 55 /
cv2 48 / exp 39 / exp2 68 / dates 34 / biz 41 / az 50, 0 ошибок консоли. Превью
пересобрано с `ef5fd71` по той же ссылке. Мерж/деплой не выполнялись, миграций нет.
ROC-страницы по-прежнему HTTP 403 — непроверенные правила Arizona перечислены в отчёте.

2026-09-05, позже (запись Claude): по сводной спецификации владельца от 2026-09-05
(CHANGES_REQUESTED, 12 разделов) выполнены три малых коммита в ветке
`claude/state-specific-intake-v3`: `01dc5a1` (Месяц + Год двумя полями, даты без
сдвига зоны, время/место экзаменов, заметки), `1ba33e0` («Для кого лицензия?»,
DBA в подготовке заявления, «Ваш путь», одна сводка + «Все ответы», палитра),
`755f49a` (правила штата в `ROADMAP_RULES[state]`, Arizona из статутов azleg.gov,
`roadmap-az.html`, контекст «штат · классификация», перенос периодов с
подтверждением, перепроверка после смены классификации). Discovery report — до
изменений. Отчёт: `tasks/reports/2026-09-05-consolidated-roadmap-corrections.md`.
`test-roadmap-v3.mjs` 138/138, `verify.js` чист, Playwright dates 34 / biz 40 /
az 50 / регрессия 52+48+39+68. Превью пересобрано с `755f49a` по той же ссылке.
Мерж/деплой не выполнялись, миграций нет; `main` осн. репо остаётся `c2e9dd6`.

2026-09-05 (запись Claude): по спецификации владельца от 2026-09-05 (CHANGES_REQUESTED
по результатам проверки preview) Experience Builder скорректирован: уровень работы
отделён от формы занятости по форме CSLB 13A-11, карточка периода — три группы без
перестроения, свёрнутые карточки, частичная занятость отдельно без выдуманного
пересчёта, подтверждающий/документы по официальному списку, прокрутка измерена и два
дефекта исправлены. Ветка `claude/state-specific-intake-v3` @ `0967abf`.
Отчёт: `tasks/reports/2026-09-05-experience-builder-rework.md`. Превью обновлено по той
же ссылке; запись экрана и скриншоты переданы владельцу в чате. Мерж/деплой не
выполнялись, миграций нет; `main` осн. репо остаётся `c2e9dd6`.

2026-09-04 (запись Claude): по отдельной спецификации владельца от 2026-09-04
(17 разделов) анкета переработана из набора экранов с кнопками «Далее» в
последовательную беседу: один вопрос за раз, отвеченные блоки сворачиваются в
строки «✓ ответ · Изменить», правка любого ответа пересчитывает ветку и
каскадно удаляет ставшие неактуальными ответы; тип заявления выводится
карточкой результата; держатель лицензии — 5 вариантов, включая «пока не знаю»
(в план добавляется этап выбора формы бизнеса); опыт — компактные карточки
периодов и собственный выбор месяца и года; образование, военная служба и этап
заявления разведены по отдельным блокам; черновик анкеты переживает
перезагрузку. Ветка `claude/state-specific-intake-v3` @ `277a993`.
Отчёт: `tasks/reports/2026-09-04-intake-v3-conversational.md`.
Прогнаны 20 сценариев спецификации, `scripts/test-roadmap-v3.mjs` 99/99,
`scripts/verify.js` без ошибок. Превью обновлено по той же ссылке.
Мерж/деплой не выполнялись, миграций нет; `main` осн. репо остаётся `c2e9dd6`.

2026-08-31, позже (запись Claude): замечания владельца (CHANGES_REQUESTED —
прокрутка в Experience Builder и tribal в основном списке типов бизнеса)
исправлены. Ветка `claude/state-specific-intake-v3` @ `a42b380`
(`c09b649` — сами исправления, `a42b380` — 12 исправлений по
состязательному ревью диффа, включая две блокирующие находки: подмену
экрана под кликом при изменении состава активных экранов и потерю локально
сохранённого роадмапа из-за несимметричных ключей чтения/записи).
Отчёт: `tasks/reports/2026-08-31-intake-v3-changes-requested-fixes.md`.
Превью обновлено по той же ссылке. Мерж/деплой не выполнялись, миграций нет.

2026-08-31 (запись Claude): реализовано и собрано превью.
Ветка `claude/state-specific-intake-v3` @ `4bfd7d7` (база — версия роадмапа,
которая сейчас на ревью, `a8553ba`). Мерж НЕ выполнен, миграции НЕ применялись,
деплоя НЕ было; `main` осн. репо остаётся `c2e9dd6`.

Отчёт по требуемому формату (discovery, изменённые области, модель данных,
результаты тестов, официальные источники, превью, открытые вопросы,
состояние production): `tasks/reports/2026-08-31-state-specific-intake-v3.md`.
Превью: https://claude.ai/code/artifact/5c1a7ef2-4ed3-420a-af44-342ff6a3602a

Кратко: штат приходит со страницы и больше не спрашивается; классификация
обязательна и «не уверен» как ответа нет; история лицензий и роль на ней
спрашиваются в начале; тип заявления выводится и подтверждается; владелец при
действующем RME получает отдельную ветку, где владение явно отделено от
квалифицирующего опыта и от статуса квалифайера; опыт собирается периодами с
защитой от двойного счёта; образование не вычитается из четырёх лет; waiver
подаётся только как возможное рассмотрение по BPC 7065.1; Law & Business не
гасится автоматически; выбытие квалифайера даёт первоклассную дату по
BPC 7068.2; ключи хранения несут штат и классификацию, поэтому роадмапы разных
штатов больше не затирают друг друга; календарная арифметика переведена целиком
в UTC.

Проверки: 77 логических тестов, 22 render-проверки анкеты и 11 сценариев,
10 проверок веток waiver и плана экзаменов, 16 проверок превью, паритет
EN/ES/RU 474/474/474, verify.js без ошибок, 360px без горизонтального скролла.

Открытые вопросы помечены `UNKNOWN` в отчёте: признаваемые классификации по
штатам реципрокности, официальные формы для add-classification и
replace-qualifier, Невада и Аризона (остаются недоступны намеренно), объём
зачёта военной подготовки, загрузка доказательств (требует отдельного
security review).

## Objective

Redesign only the LICENA Roadmap questionnaire logic and produce a
non-production preview for owner review.

Do not redesign the entire site. Preserve the existing LICENA visual system.
Do not merge, deploy, apply migrations, or change production.

This task supersedes the current fixed six-screen questionnaire design where
the requirements below conflict with it.

## Owner decisions

1. A state-specific Roadmap page already determines the licensing state.
   Do not ask the state again inside that Roadmap.
2. A user may maintain separate Roadmaps for multiple states.
3. Classification selection is mandatory. Do not allow Roadmap generation
   without a classification.
4. Do not include `I'm not sure` as a classification answer. A separate
   “Compare classifications” tool/link may exist before the Roadmap, but it is
   not an answer and does not create a Roadmap.
5. The primary journey is for the user personally. Do not lead with a question
   about qualifying a license for somebody else.
6. Existing license history must be asked near the beginning because it can
   materially change application type, required exams, experience review, and
   waiver guidance.
7. Business/entity and qualifying-individual concepts must remain separate.
8. DBA/FBN is not an entity type.
9. Experience must become a structured Experience Builder rather than only an
   approximate-years multiple choice.
10. Every state must use its own official requirements, terminology, forms,
    exams, deadlines, waivers, reciprocity, issuance, and Ready-to-Work logic.
    Do not copy California logic and rename the agency.
11. Licensing reminders are not part of the mandatory intake while delivery is
    not implemented. Remove the reminders screen from the required path.

## Mandatory safety and legal-content rules

- Inspect the current private implementation before changing it.
- Work on a separate feature branch from the currently reviewed Roadmap base.
- Prefer backward-compatible changes and preserve saved Roadmaps.
- Do not break auth, payments, courses, Supabase, analytics, or production
  routes.
- Verify California rules against current official CSLB/government sources
  before coding.
- Mark unsupported or unverified state rules `UNKNOWN`; do not invent them.
- Do not make legal eligibility determinations.
- Do not claim that LICENA submits applications to a state agency.
- Do not collect SSN, ITIN, driver license number, date of birth, fingerprint
  records, or other highly sensitive fields.
- Employer/company names, project details, certifier details, and supporting
  evidence must never be sent to marketing analytics.
- Do not publish private code, secrets, PII, or private implementation details
  in `licena-docs`.
- Only the owner can set `APPROVED_FOR_MAIN`.

## State context and multi-state persistence

Design the Roadmap so state is supplied by route/page context, for example:

- California page → California Roadmap;
- Nevada page → Nevada Roadmap;
- Arizona page → Arizona Roadmap.

The same user must be able to have independent progress for multiple states.
Audit the current persistence key/model and propose a collision-safe identity
that includes at least state and classification/path as appropriate.

Do not silently overwrite a California Roadmap with Nevada or Arizona answers.
If a generic entry page exists, it may ask which state to open; the
state-specific questionnaire must not ask again.

For this iteration, implement only states whose official logic and product
support are confirmed. Unsupported state previews must say unavailable rather
than showing California requirements.

## Required intake order

### 1. Classification

Prompt:

> Which license classification are you applying for?

- Required exact classification.
- Fix the current broken `{name} trade exam` interpolation.
- No `I'm not sure` option.
- Do not continue or generate a Roadmap until selected.
- A separate “Compare classifications” link may leave/open a comparison flow.

### 2. Existing license history

Ask whether the user currently holds or previously held a contractor license
in this state:

- active;
- inactive;
- expired/suspended;
- previously served as qualifier;
- no license history.

If applicable, collect only non-sensitive planning fields:

- optional license number;
- current classifications;
- current status;
- current role on the license;
- whether this is an existing company license.

### 3. Current role on an existing company license

Required options where applicable:

- current qualifying individual;
- owner/officer/member, but an RME or other qualifier qualifies the license;
- previously qualified the license;
- employee of the licensed company.

If the user is an owner/officer/member while an RME qualifies the license,
ask the goal:

- replace the RME and personally qualify the company;
- keep the RME and add a classification;
- obtain a separate license personally;
- prepare in case the RME leaves.

Do not treat ownership alone as qualifying experience or qualifier status.

### 4. Application type

Derive or confirm the correct state-specific path:

- first/original license;
- add classification to an existing license;
- replace qualifying individual;
- new license for another entity;
- previous/reapplication path;
- reciprocity from another state.

Do not show a first-license Roadmap to an add-classification or replace-RME
user.

### 5. Who will hold the license

Ask:

- me as Sole Owner;
- an existing business;
- a business I plan to create.

For Sole Owner, do not ask whether the entity has been formed.

For an existing business, collect:

- entity type;
- exact legal name;
- state of registration;
- registration/good-standing status;
- DBA/FBN separately.

For a planned business, record intent only. Do not build company formation.

Only in a company path, and only when needed later, ask whether the user will
personally qualify the company. Do not make this an early standalone screen for
the normal personal journey.

## Experience Builder

Replace the single approximate-years answer with repeatable experience periods.

For each period collect:

- employer/company label;
- start month/year;
- end month/year or current;
- employee / self-employed / owner;
- state-specific role/level;
- classification/type of work;
- concise work description;
- full-time/part-time where officially relevant;
- whether a person with firsthand knowledge can certify the period;
- whether supporting records exist.

Allow multiple employers/periods. Detect overlapping periods and do not
double-count overlapping calendar time.

Provide an evidence-readiness view:

- claimed calendar duration;
- potentially relevant periods;
- overlap warning;
- period without certifier;
- period without supporting records;
- remaining facts to review.

Never output “eligible,” “approved,” or “you meet the legal requirement.”
Use cautious language such as:

> You entered approximately X years of potentially relevant experience. The
> licensing agency determines whether the experience qualifies and may request
> supporting documentation.

Do not upload evidence in this task. Only record whether categories of evidence
exist. Detailed document upload/storage requires a separate security review.

## Education, apprenticeship, and military experience

For California, include a conditional section for education/training that may
be reviewed for experience credit:

- education/training type;
- field/program;
- institution country;
- completed/not completed;
- completion date;
- official transcript available;
- apprenticeship certificate available;
- foreign credential translated/evaluated.

Do not automatically subtract education credit from required experience.
Explain that CSLB determines any credit after reviewing official documentation.

Include military experience/training only where verified by official state
sources and keep it separate from generic education.

## California: company licensed through an RME

Create a distinct branch for an owner/officer/member whose company license is
currently qualified by an RME.

The Roadmap must distinguish:

1. ownership of the company;
2. actual qualifying experience;
3. current qualifier status.

Possible results:

### Insufficient documented experience

Show an experience-building/evidence plan while the existing RME remains the
responsible qualifier. Do not imply that time as an owner automatically counts.

### Four years of potentially qualifying, certifiable experience

Show a preparation path for replacing the qualifying individual, including
experience review, the applicable application, fingerprints where required,
and examinations unless CSLB confirms a waiver.

### Possible examination waiver

Never say that waiting five years produces an automatic waiver.

Only show a possible-waiver review when the verified official conditions may
apply. State clearly that waiver approval is discretionary and determined by
the Registrar.

The family-business waiver is a narrow path, not a general owner/RME path. It
requires the verified conditions, including immediate-family relationship,
active engagement for the required period, same classification, continuation
of the family business following absence/death, experience documentation, and
Registrar discretion.

If the current RME disassociates, track the official replacement deadline as a
first-class important date. Verify the exact rule and source before coding.

## Exam logic

Track Law & Business and Trade separately.

Do not automatically mark Law & Business unnecessary merely because the user
or company already has a license.

Determine the guidance branch from verified facts such as:

- application type;
- state;
- classification;
- current/previous qualifier status;
- when relevant examinations were passed;
- existing license status/good standing;
- reciprocity conditions;
- state agency determination/notice.

Use “may qualify for waiver” until officially confirmed by the agency.

## Application and later stages

Preserve adaptive state-specific branches for:

- application preparation/submission;
- review/corrections;
- fingerprinting;
- Law & Business exam;
- Trade exam;
- final issuance requirements;
- license issued;
- Ready to Work.

Submission does not mean classification or experience was approved.

All important dates remain first-class data. Fix the previously identified
calendar bug: date-only values must not mix UTC parsing with local
`getDate/setDate/getMonth/setMonth`. Test 90-day and month-based deadlines
across DST boundaries, leap years, and end-of-month dates.

## Preview scenarios

Create a non-production preview and screenshots for at least:

1. first-time California Sole Owner;
2. first-time applicant with an existing LLC;
3. owner of a licensed corporation whose RME currently qualifies it;
4. same owner with less than four years documented experience;
5. same owner with four years of potentially certifiable experience;
6. possible family-business waiver review;
7. add-classification path;
8. reciprocity/other-state license path;
9. returned-for-correction path;
10. issued-license/Ready-to-Work path;
11. two independent state Roadmaps for one synthetic user.

Preview data must be synthetic.

## Verification

Run and report:

- existing verification script;
- tests/typecheck/build available in the repository;
- EN/RU/ES key parity;
- adaptive branch tests for all preview scenarios;
- backward compatibility with existing saved Roadmaps;
- multi-state persistence collision test;
- no-classification/no-Roadmap test;
- Sole Owner does not receive entity-formation questions;
- RME owner is not treated as qualifier;
- ownership time is not automatically counted as qualifying experience;
- overlapping experience is not double-counted;
- education credit is not automatically granted;
- Law & Business is not automatically waived;
- submission does not approve experience/classification;
- calendar deadline tests around DST/leap/end-of-month;
- mobile 360px and desktop screenshots;
- keyboard-accessible flow.

## Required handoff in licena-docs

Publish only:

- status;
- discovery summary;
- private branch and commit SHA;
- changed areas;
- data-model/migration summary;
- tests/build results;
- official sources used;
- preview URL;
- screenshots;
- unresolved legal/product questions;
- explicit production state.

Do not copy private source code or PII.

## Exit status

After implementation and preview: **READY_FOR_REVIEW**.

Do not merge or deploy. Do not apply migrations. Stop and wait for owner review.
