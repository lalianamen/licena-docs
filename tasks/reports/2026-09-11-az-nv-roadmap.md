# Отчёт: Arizona и Nevada License Roadmap — рулсеты, страницы, тексты (READY_FOR_REVIEW)

Последняя сверка: 2026-09-11
Статус: **READY_FOR_REVIEW** — ветка осн. репо `claude/state-specific-intake-v3` @ `eea5c20` (поверх релиза `66d1727`); в `main` **НЕ смержено**, деплой не выполнялся, миграций нет. Оба штата остаются `available:false` (`ROADMAP_STATES.az/.nv`, `ROADMAP_RULES.az/.nv.available`) — страницы `roadmap-az.html` и `roadmap-nv.html` на сайте показывают экран «штат пока недоступен» до решения владельца.
Запрос владельца (2026-09-11): «теперь нужно проанализировать аризону и неваду и на базе проведенного анализа подготовить родмап и туда».
Аналитическая база: `tasks/reports/2026-09-11-az-nv-roadmap-analysis.md` (источники, даты чтения, `UNKNOWN`).

## 1. Что сделано

| Область | Arizona (ROC) | Nevada (NSCB) |
|---|---|---|
| Порядок шагов | **Экзамены до заявления** (PSI bulletin 2477 «Licensing applications cannot be accepted until all examination requirements have been completed»; A.A.C. R4-9-106 — результат действителен два года). Порядок: classification → experience → prep_law_business (SRE) → prep_trade → schedule_pass_exams → prepare_application → submit_application → fingerprinting → application_review → final_requirements → license_issued | Порядок Калифорнии сохранён (заявление → проверка опыта / exam eligibility → CMS + trade → бонд, сбор, страхование → выдача) — так описывает процесс NSCB (License Requirements, License Examinations) |
| Движок | новый флаг `R.examsFirst` (`derive` — экзамены «живые» без принятого заявления; `recommendNext` — ветка «сначала сдайте экзамены»; блок анкеты `exams`; в блоке заявления экзамены не спрашиваются повторно; ответ «заявление не подано» не стирает статусы экзаменов) | без изменений движка: рулсет ложится на существующие шаги |
| Факты в конфиге | `ROADMAP_AZ_EXAM_FACTS` (70 %, SRE $61, trade $66/$116, solar-only $40, срок оплаты 1 год, пересдача через 30 дней, 3 попытки, затем 90 дней, заявление ≤ 2 лет после сдачи), `ROADMAP_AZ_FEES` (R4-9-130: general commercial $200 + $580 …; recovery fund $370 / $270), `ROADMAP_AZ_BONDS` (R4-9-112 по классу и объёму), `ROADMAP_AZ_PROCESSING` (60/40/30 дней), `military.creditPossible` (R4-9-119) | `ROADMAP_NV_*`: опыт 4 года из 15 (NRS 624.260), уровни journeyman/foreman/supervising/contractor, зачёт обучения ≤ 3 лет, endorsement 12 штатов / 4 года / trade-экзамен снимается, CMS всегда (NSCB), экзамены PSI 270 (CMS 60 вопросов / 120 мин / 45 = 75 %, open book; $95 / $140; 3 попытки, 14 дней, после третьей — заявление void, новое через 30 дней), сборы $300 / $600 на 2 года (NRS 624.280 — предельные), бонд $1 000–$500 000 (NRS 624.270, NSCB), recovery fund по лимиту (NRS 624.470), финансовые пороги, отпечатки/background (NRS 624.265, NAC 624.681), страхование (NRS 624.256), RTW: коммерческие автомобили (NRS 624.288) |
| Страница | `roadmap-az.html` (существующая; версии скриптов) | **новая** `roadmap-nv.html` (`data-state="nv"`) |
| Кабинет | `js/app-cabinet.js`: кнопка roadmap ведёт на `roadmap-az.html` / `roadmap-nv.html` по выбранному штату (иначе `roadmap.html`) | то же |
| Тексты (`js/roadmap/i18n-roadmap.js`) | оверлей `_az` 163 ключа × EN/ES/RU (перестроены шаги под exams-first: `nx_prep_t/b`, `qExamsH/Sub`, `t_/d_/todo_/docs_*` для prep/exams/application/final) | новый оверлей `_nv` 171 ключ × EN/ES/RU; база +11 ключей (`rowExams`, `qExamsH/Sub`, `recipState_*` для штатов endorsement) |
| Логика ожиданий | `roadmap-logic.js`: кандидат на освобождение от экзаменов `prior_qi_4y` (Nevada) рядом с `prior_qp_5y` | — |

Паритет ключей: база 823 = 823 = 823; `_az` 163/163/163; `_nv` 171/171/171 (EN/ES/RU).

## 2. Ошибки, найденные тестами в этом раунде и исправленные до коммита

1. **Ответ «заявление не подано» стирал статусы экзаменов** (`buildStage`, ветка `a_no`) — в Калифорнии экзамены существуют только после принятия заявления, в Аризоне они сдаются до него. Теперь для `examsFirst` поля `lawStatus/tradeStatus/lawDate/tradeDate/lawWhere/tradeWhere` сохраняются. Тест: `az-suite2` A8.
2. **Базовая строка `expOutsideNote` содержала «10-year window» буквально** — при 15-летнем окне Невады расчёт был верным (месяцы вне окна считались по `withinYears: 15`), а подпись врала. Переопределено в `_nv` (EN/ES/RU): «15-летнее окно». Тест: `nv-suite` C3.
3. **Шесть базовых строк с формулировками CSLB были достижимы на страницах AZ/NV**: `nsFlag` (классификация «не уверен»), `expDocTypesL`, `expNonLicNote`, `expJlNote`, `expCertRelNote` (Experience Builder), `nx_susp_b` (снятие приостановки); для NV дополнительно `att_reciprocity` (endorsement). Переопределены в `_az` и `_nv` × 3 языка с формулировками из источников штата (NRS 624.260(8) для journeyman; форма Certification of Work Experience NSCB — часть 2 заполняет работодатель, подпись под страхом ответственности за лжесвидетельство; A.R.S. 32-1122(B) / A.A.C. R4-9-119 для Аризоны). Оставшиеся базовые строки с «CSLB» на AZ/NV недостижимы: они привязаны к правилам, которых у этих штатов нет (`R.expired`, `correctionDays`, `examWindowMonths`, `wcAlways`, `classPages`, `assistant`, `finalItems.asbestos`), либо не используются кодом (`qAppL`, `qInstrL`, `stage_accepted`, `qFingerStL`, `retakeNote`, `qExpSub3`, `att_renew`) — проверено grep по `app-roadmap.js` и `roadmap-logic.js`.

## 3. Проверки

- `node scripts/verify.js` — 0 ошибок (139 файлов `node --check`, банки, offer, course-ref).
- `node scripts/test-roadmap-v3.mjs` — **200/200** (было 183): три устаревшие проверки Аризоны переписаны под exams-first и источники A.A.C./PSI, добавлены проверки фактов AZ (R4-9-106/130/112) и 12 проверок рулсета NV.
- Playwright (Chromium, `http://127.0.0.1:8901`, доступность AZ/NV включается перехватом `roadmap-config.js` в тесте — репозиторий не трогается):
  - `az-suite2.mjs` — **43/43**: анкета с блоком экзаменов до заявления, обе сдачи → следующий шаг больше не экзамены, порядок карточек, источники только azleg / law.cornell.edu (A.A.C.) / PSI, суммы и сроки на карточках, ключ хранения `lp:roadmap:az:…`, экран «недоступно» без переключателя, RU 360 (анкета + план, без горизонтального скролла), ES план без плейсхолдеров и без CSLB/California.
  - `nv-suite.mjs` — **53/53**: список классификаций NV, вопрос endorsement (12 штатов), стадии NSCB, оба экзамена после eligibility, вопрос об отпечатках в словах NSCB, план в порядке Калифорнии, источники только nvcontractorsboard / leg.state.nv.us / PSI с датой 2026-09-11, CTA на `nv-cms` / `nv-b`, Experience Builder (уровни journeyman/foreman/supervising/helper; contractor — только с историей лицензии, как в CA; 15-летнее окно; форма Certification of Work Experience), независимость от California (перенос периодов предлагается, не копируется), экран «недоступно», RU 360, ES.
  - регрессия California на том же дереве: undo 56 / final 42 из 43 / db-roundtrip 8 / expq 49 из 50 / cv 53 / exp2 68 / biz 41 / dates 34 / bugs 26 из 27 / exp-logic 27 (`8-az` в final-suite, `G-az` в expq-suite и `G` в bugs-suite ожидаемо падают с 2026-09-11: страница AZ на репозиторном конфиге недоступна).
  - превью (собранный файл, `file://`): `pv-check.mjs` 9/9 — `?state=nv`, `?state=az`, CA по умолчанию, без ошибок консоли.
- Скриншоты (scratchpad сессии): `az2-roadmap-en.png`, `az2-passed-en.png`, `az2-ru-360.png`, `az2-ru-360-plan.png`, `az2-es-plan.png`, `nv-roadmap-en.png`, `nv-exp-en.png`, `nv-ru-360.png`, `nv-ru-360-plan.png`, `nv-es-plan.png`, `pv-nv.png`, `pv-az.png` — просмотрены; в репозиторий не добавлялись.

## 4. Превью

- `preview/roadmap/licena-roadmap-v3.html` пересобран с `eea5c20`: в переключателе появились «AZ · … (превью, не опубликовано)» и «NV · … (превью, не опубликовано)»; **только в превью** после конфига вставлен скрипт, включающий `available` для AZ/NV; встроена актуальная `js/paths.js` (в старой копии не было классификаций Невады).
- Внешняя ссылка по ветке `main` licena-docs: `https://raw.githack.com/lalianamen/licena-docs/main/preview/roadmap/licena-roadmap-v3.html?state=nv` (и `?state=az`); постоянная ссылка по коммиту — в `preview/roadmap/README.md`.
- Claude-артефакт «Roadmap Intake V3»: перепубликация в этом раунде не выполнена (сервис требует полного чтения ранее опубликованной версии перед публикацией); внешняя ссылка выше актуальна.

## 5. UNKNOWN и открытые вопросы (для владельца)

| Вопрос | Состояние |
|---|---|
| Arizona: страницы ROC (roc.az.gov / azroc.gov) | HTTP 403 из песочницы — факты только со страниц ROC (форма опыта и её подписанты, требования trade-экзамена по классу RC-L-206B, практика отпечатков, момент оплаты license fee, штаты reciprocity) остаются `UNKNOWN`; в конфиге `reciprocity: null`, `classPages: null`, `correctionDays: null` |
| Arizona: A.A.C. читался через зеркало law.cornell.edu | номера разделов и history notes совпадают с официальными; официальный apps.azsos.gov — 403 |
| Nevada: точная сумма за отпечатки, сроки рассмотрения | `UNKNOWN` (NSCB публикует «fee set by DPS/FBI»; сроков на страницах нет) |
| Nevada: окно опыта | NSCB FAQ — «10 years», NRS 624.260(6) — 15 лет; в конфиге статут (15), расхождение отмечено в аналитике |
| Уровень «contractor» в Experience Builder | показывается только при истории лицензии (логика унаследована от CA: `uiLevels.filter(k => k !== "contractor" \|\| licHist(a))`) — для NV это совпадает с формой (Contractor — вариант связи с работодателем), решение не менялось |
| Публикация | по решению владельца: для каждого штата отдельно `available:true` в `ROADMAP_STATES` и `ROADMAP_RULES` + bump `roadmap-config.js?v=` на четырёх страницах; входы на лендинге/в кабинете под AZ/NV не добавлялись (кабинет уже маршрутизирует по штату) |

## 6. Изменённые файлы (осн. репо, `eea5c20`)

`js/roadmap/roadmap-config.js` (v15), `js/roadmap/app-roadmap.js` (v23), `js/roadmap/roadmap-logic.js` (v9), `js/roadmap/i18n-roadmap.js` (v21), `js/app-cabinet.js`, `roadmap.html`, `roadmap-az.html`, `application.html` (bump), **новый** `roadmap-nv.html`, `scripts/test-roadmap-v3.mjs`.

## 7. Риски

- Без ROC-страниц Аризона описана по статуту, A.A.C. и бюллетеню PSI — формы/инструкции ROC могут добавлять требования (в текстах это сказано явно: «ROC's instructions govern»).
- `examsFirst` — новый режим движка; покрыт `az-suite2` (43) и node-тестами; California-регрессия прогнана на том же дереве.
- Оверлеи большие (163 + 171 ключей × 3 языка); паритет ключей проверен скриптом, тексты RU/ES написаны по EN и источникам, независимой вычитки не было.

## Source References

Осн. репо (ветка `claude/state-specific-intake-v3` @ `eea5c20`): `js/roadmap/roadmap-config.js`, `js/roadmap/app-roadmap.js`, `js/roadmap/roadmap-logic.js`, `js/roadmap/i18n-roadmap.js`, `js/app-cabinet.js`, `js/paths.js`, `roadmap.html`, `roadmap-az.html`, `roadmap-nv.html`, `application.html`, `scripts/test-roadmap-v3.mjs`, `scripts/verify.js`. Официальные источники — по списку в `tasks/reports/2026-09-11-az-nv-roadmap-analysis.md` (§0 и Source References). Тестовые сценарии Playwright — scratchpad сессии (`az-suite2.mjs`, `nv-suite.mjs`, `state-lib.mjs`, `pv-check.mjs`, регрессионные сюиты), в репозитории не хранятся.

## Verification Status

**Partially Verified** — код, тексты и тесты проверены чтением и прогоном; позиции `UNKNOWN` в §5 не проверены по первоисточнику (ROC недоступен, NSCB не публикует).
