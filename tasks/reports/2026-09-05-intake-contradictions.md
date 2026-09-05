# LICENA License Roadmap — исправление противоречий intake и проверка licensing roadmap (отчёт для владельца)

Последняя сверка: 2026-09-05
Статус: **READY_FOR_REVIEW** (исходный статус спецификации — CHANGES_REQUESTED).
Merge в `main` и production deploy НЕ выполнялись — только после явного `APPROVED_FOR_MAIN`.
`main` основного репозитория на момент отчёта — `9a8a577`; ветка раунда в него не входит.

## 1. Ветка и коммит

Репозиторий `lalianamen/llicena`, ветка `claude/state-specific-intake-v3`, запушена.
База раунда — `755f49a` (Consolidated Roadmap Corrections). Один коммит — `ef5fd71`.

## 2. Аудит до изменений (база `755f49a`)

Матрица «вопрос → условие показа → сохраняемые поля → зависимые вопросы → влияние на план»
построена по `UNITS` в `js/roadmap/app-roadmap.js` и по `deriveAppType` / `experienceReadiness` /
`examPlan` в `js/roadmap/roadmap-logic.js`. Воспроизведение шести проблем (Playwright, до правок,
файл `bugs-before.out` в scratchpad):

| # | Что воспроизведено | Причина в коде |
|---|---|---|
| A | «Никогда не было лицензии» + уровень периода «Contractor» — уровень принимается без уточнения, `experienceReadiness` считает его как journey-level | уровень не сверялся с историей лицензий |
| B | `h_active` → `companyLicensed=y` → смена истории на «никогда»: `companyLicensed` остаётся; при holder «существующая компания» `deriveAppType` даёт `add_class` | `hist` не чистил зависимые поля; `deriveAppType` не требовал реальной лицензии |
| C | номер существующей лицензии (`licenseNumber`) и номер выданной делили одно поле; скрытие блока «выдана» удаляло его через `prune()` | одно поле на два смысла |
| D | `applyStage()` при любой смене стадии/`examWhich` писал `lawStatus = tradeStatus = x_not` — «Trade назначен» стирал «Law сдан» | статусы зависели от стадии |
| E | любая лицензия другого штата → путь reciprocity, блок опыта скрыт (`needsExperience=false`), trade «may_waive» | `deriveAppType` проверял только `otherStateLicense==="y"` |
| F | стадия «выдана» писала `lawStatus = tradeStatus = x_pass`; план показывал «✓ Completed» для обоих экзаменов | выдача и результаты экзаменов не разделены |

## 3. Что исправлено (`ef5fd71`)

| # | Исправление | Где | Проверка |
|---|---|---|---|
| A | Период с уровнем «Contractor» при отсутствии истории лицензий (в выбранном штате и в других) сохраняется, помечается «уточните историю лицензий», не входит в оценку journey-level; в карточке — заметка без обвинений и кнопка «Обновить историю лицензий» (`editUnit("hist")`) | `experienceReadiness(…, {licenseHistory})` → `periodsLevelConflict`; `expLevelConflict*`, `expContractorNote/Fix` | `bugs-suite` A1–A2; node-тест «contractor conflict» |
| B | При «никогда не было» удаляются `licenseNumber`, `histClasses`, `companyLicensed`, `expiredWithin`, `appTypeConfirmed`; role/goal/waiver уходят через `prune()`; смена role/goal/recip сбрасывает `appTypeConfirmed`; `deriveAppType` даёт add_class / replace_qualifier / renew / clear_suspension только при реальной лицензии | `hist` unit, `deriveAppType(a, cfg)` | B0–B2; node «stale role/companyLicensed → original» |
| C | `licenseNumber` (существующая лицензия) и `newLicenseNumber` (выданная) — два поля; миграция v4 переносит старое значение в `newLicenseNumber` только когда история = «никогда» и лицензия выдана (однозначно), иначе оставляет как есть; `dbSave` пишет `newLicenseNumber || licenseNumber` | `migrate()`, `issuedX` unit | C0–C2 |
| D | Стадии заявления: `waiting / corr / acc / issued / unknown`; при «принято» — отдельный статус для Law & Business и для Trade (`x_not / x_sched / x_pass / x_fail`, + `x_wreq / x_waived` для кандидата на освобождение), у каждого своя дата и место; `applyStage()` меняет только стадию; `examWhich` / `passWhich` / `stg_exam` / `stg_p1` / `stg_pall` удалены, старые значения нормализуются миграцией | `examBlock()`, `applyStage()`, `migrate()` | D1–D4 (passed+scheduled, passed+failed, failed+passed; план показывает каждый экзамен отдельно); `dates-suite` B |
| E | `reciprocityApplies(a, rec)` → `{ok, reasons: state / years / good}`; reciprocity только при штате из списка CSLB, ≥5 лет активной лицензии и good standing; иначе обычная первая лицензия с объяснением, какое условие не выполнено, без обещаний; опыт спрашивается всегда (`needsExperience` включает reciprocity); trade — «может быть освобождён», Law & Business остаётся | `roadmap-logic.js`, `recip` unit (`recipMetNote` / `recipNotMetNote`) | E1–E5; node «applicability / reasons / examPlan» |
| F | «Лицензия выдана» пишет только `issued=y`; результаты и даты экзаменов не придумываются; в плане экзамены без записи показаны как «—»; выдача ≠ «готов работать» — чек-лист остаётся | `applyStage("stg_issued")`, `exCell` | F1–F3 |

Дополнительно по спецификации: классификации без trade-экзамена (`az-any`) не спрашивают его статус,
финальные требования открываются после единственного требуемого экзамена (G1–G2); статусы «waiver
запрошен» / «waiver подтверждён агентством» показываются только кандидату, ревью говорит «может
подойти», никогда «освобождён»; неподтверждённый waiver не открывает финальные требования (H1–H4).

## 4. Правила CSLB — источник, дата, применимость, событие

Все страницы прочитаны 2026-09-05 (`curl`), значения записаны в `roadmap-config.js` с `verified: "2026-09-05"`.

| Правило | Источник | Применимо когда | Что открывает / завершает |
|---|---|---|---|
| Reciprocity: AZ, LA, MS, NV, NC; лицензия активна и в good standing «for the previous five years»; форма Request for Verification of License; подаётся Application for Original Contractor's License; список классификаций по штату — на странице CSLB (в продукте UNKNOWN) | cslb.ca.gov/Contractors/Applicants/Reciprocity/Reciprocity_Requirements.aspx | штат ∈ списка ∧ ≥5 лет ∧ good standing | путь «первая лицензия с reciprocity» |
| Trade exam: «CSLB may waive the trade portion … retains the right to require»; business law по-прежнему требуется | …/Reciprocity/Reciprocity_Exam_Requirements.aspx | reciprocity применима | trade = «может быть освобождён», Law = требуется |
| Law & Business при reciprocity требуется, если не сдан за последние 5 лет | форма 13A-1 (rev. 01/2026), Q17 | `lawPassedWithinYears = y` | Law = «может быть освобождён» |
| Заявление принято: Live Scan packet + Notice to Schedule (PSI); 18 месяцев на сдачу; 21 день между попытками; результаты действительны 5 лет; пересдаётся только проваленный | …/Exam_Application/Application_Accepted.aspx | стадия «принято» | экзамены и отпечатки параллельно |
| Возврат на исправление: 90 дней, иначе void; сбор не возвращается | …/Exam_Application/Application_Returned.aspx | стадия «исправление» | срок исправления |
| Live Scan: после posting; третья копия в CSLB в течение 90 дней; $49 + rolling fee | …/Exam_Application/Get_Fingerprinted_-_Live_Scan.aspx | подано ∧ не отклонено | шаг fingerprinting |
| Contractor's Bond $25 000 (BPC 7071.6); Bond of Qualifying Individual $25 000 при RME или RMO <10 % voting stock, иначе exemption certification; tribal — exempt (BPC 7071.9) | …/Bond_Information/Bond_Requirements.aspx | bond — всем; QI bond — не sole owner | финальные требования |
| LLC: $100 000 employee/worker bond (7071.6.5) + $1M liability (7071.19); QI bond | cslb.ca.gov/About_Us/LLC.aspx | entity = LLC | финальные требования |
| Сборы: Original Application $450; Initial License $200 sole / $350 non-sole; Additional Classification $230; Replacing the Qualifier $230 | cslb.ca.gov/About_Us/Library/Fees.aspx | всем | финальные требования (fee) |
| Истёкшая: renew в течение 5 лет после истечения; позже — Application for Original License; работа с истёкшей лицензией = без лицензии | …/Renew_License/Failing_To_Renew_Your_License.aspx | `h_expired` + «меньше / больше 5 лет» | путь renew / reapply |
| Приостановка должна быть снята до возврата в good standing | …/Renew_License/General_Renewal_Information.aspx | `h_suspended` | путь clear_suspension |
| Экзамены: Law and Business + Trade «except C-61»; waivers «under certain conditions»; asbestos open-book после сдачи | cslb.ca.gov/Contractors/Applicants/ | trade не требуется, если у пути нет trade-курса | схема статусов |

Различия, отражённые в продукте: original / add classification / replace qualifier / renew / reapply /
clear_suspension / reciprocity; submitted / returned / accepted / issued; «может быть освобождён» ≠
«запрошен» ≠ «подтверждён агентством» ≠ фактический результат; владение компанией с RME не даёт
личного опыта (level conflict); «прошло N лет → лицензия» нигде не выводится; финальные требования
не закрываются двумя экзаменами или одним бондом — каждый применимый пункт подтверждается отдельно.

## 5. Arizona — непроверенное (roc.az.gov HTTP 403 при `curl` и WebFetch, 2026-09-05)

Не проверено и НЕ реализовано (в конфиге `null`/отсутствует, в UI скрыто, калифорнийским не заменено):
порядок «экзамен до/после подачи»; названия экзаменов и вендор; форма подтверждения опыта и кто её
подписывает; сборы; background check (кроме «may require fingerprints», A.R.S. 32-1122(H)); сроки
рассмотрения (кроме 60 дней, A.R.S. 32-1124(A)); правила истёкшей/приостановленной лицензии
(в продукте для AZ: expired → reapply без вопроса о сроке; suspended → «снять приостановку» по
инструкциям органа); взаимность. Реализовано только из статутов azleg.gov (32-1122, 32-1124,
32-1127.01, 32-1152) — см. отчёт `2026-09-05-consolidated-roadmap-corrections.md`. `az-suite`
(50) подтверждает: на странице AZ нет CSLB/California-правил, прогресс CA/AZ не смешивается.

## 6. UX по спецификации

- Штат — из `data-state` страницы; ключи `lp:roadmap:<state>[:<trade>]`; неподдерживаемый штат — честный экран без правил CA (`az-suite` E1).
- Бизнес-раздел: «На кого лицензия?» (я / компания, с пояснением) → «Компания уже существует?» → тип → детали; DBA — отдельное имя на карточке подготовки заявления; универсального «кто заполняет» нет; роль уточняется только при существующей лицензии/RME.
- Опыт — периоды (месяц/год, работодатель или форма занятости, обязанности, занятость, подтверждающий, документы); образование не засчитывается автоматически.
- Изменение ответов без потери данных и без прыжка наверх: `scroll-check.mjs` — 14 замеров, все ответы Δ=0; единственное движение — ссылка «Изменить» выравнивает открытый вопрос под липкой панелью (намеренно, `editUnit`).
- План: Current Position, Your Next Action, Parallel / While You Wait, Next Important Date; Ready to Work — только применимые подтверждённые пункты; выдача ≠ активный статус.

## 7. Изменённые файлы

`js/roadmap/roadmap-logic.js` (v6), `js/roadmap/roadmap-config.js` (v11), `js/roadmap/app-roadmap.js` (v17),
`js/roadmap/i18n-roadmap.js` (v16; базовые ключи 741 × 3 языка + слой `_az` 136 × 3, паритет EN/ES/RU),
`css/roadmap.css` (v15), `roadmap.html`, `roadmap-az.html` (бампы `?v=`), `scripts/test-roadmap-v3.mjs`.
Auth, платежи, курсы, Supabase, аналитика, production-маршруты — не тронуты. SSN/ITIN, дата рождения,
номер водительского удостоверения не собираются.

## 8. Проверки

Проект — vanilla HTML/CSS/JS без сборки и без TypeScript: typecheck/build отсутствуют по природе проекта;
«build» здесь = `node --check` всех JS в `node scripts/verify.js`.

| Проверка | Результат на `ef5fd71` |
|---|---|
| `node scripts/verify.js` | 139 файлов OK; банки/офферы синхронны |
| `node scripts/test-roadmap-v3.mjs` | 155 PASS (+17: применимость reciprocity и причины; renew/reapply/clear_suspension; устаревшие role/companyLicensed → original; конфликт уровня «contractor»; `examDone`; план экзаменов) |
| Playwright `bugs-suite` (новая) | 27 PASS — A (2), B (3), C (3), D (4), E (5), F (3), G без trade-экзамена (2), H подтверждённый waiver (4), 0 ошибок консоли |
| Регрессия `cv-suite` / `cv-suite2` / `exp-suite` / `exp-suite2` | 55 / 48 / 39 / 68 PASS (сюиты обновлены под «компания → существует?», стадию «принято» + статусы и вопрос «как давно истекла»; добавлен S4b renew) |
| `dates-suite` / `biz-suite` / `az-suite` | 34 / 41 / 50 PASS — даты в двух часовых поясах, бизнес-раздел, независимость штатов, смена классификации, save/reload |
| `scroll-check.mjs` | 14 замеров, 0 прыжков на ответах (см. §6) |
| UI EN/ES/RU, desktop + 360/390 | скриншоты §10; 0 ошибок консоли (внешние шрифты блокирует прокси песочницы — отфильтровано как не-приложение) |

Недоступные проверки (честно): нет unit-фреймворка/typecheck; `ffmpeg` отсутствует — запись проверена
по прогону, не покадрово; Supabase-сохранение не прогонялось (нет сессии) — проверен localStorage-путь
и миграция сохранённых ответов; ROC-страницы недоступны.

## 9. Миграции и rollback

Новых SQL нет. Новые ответы (`newLicenseNumber`, `expiredWithin`, `lawStatus/tradeStatus` со значениями
`x_wreq/x_waived`, `qiBond`, `llcBond`, `fee`, `appTypeConfirmed`) живут в `license_roadmaps.answers jsonb`;
колонка `license_number` получает `newLicenseNumber || licenseNumber`. Клиентская миграция `intakeV=4`
(`migrate()`): `appStage` `stg_exam/p1/pall → stg_acc` со снятием `examWhich/passWhich` (статусы экзаменов
сохраняются, как были), `licenseNumber → newLicenseNumber` только при «никогда» + «выдана». Rollback =
revert `ef5fd71`; данные не теряются (старые ключи не перезаписываются деструктивно).

## 10. Превью, скриншоты, запись

Превью: https://claude.ai/code/artifact/5c1a7ef2-4ed3-420a-af44-342ff6a3602a — собрано из файлов `ef5fd71`
(self-contained; селектор сценариев, `?state=az`); проверено `preview-check3.mjs`: CA/AZ анкеты, сценарий
`exams`, ширина 360, 0 ошибок консоли.

`tasks/reports/img/2026-09-05-intake-fix-*`: `D-exam-statuses-en.png` (EN desktop, статусы по каждому экзамену),
`E-reciprocity-es-390.png` (ES 390, условие не выполнено — путь остаётся первой лицензией),
`A-level-conflict-ru-360.png` (RU 360, уровень «Contractor» без истории — заметка и кнопка),
`F-issued-en.png` (EN desktop, «выдана»: экзамены «—», чек-лист остаётся), `final-requirements-ru-390.png`
(RU 390, бонд и Bond of QI с источниками), `holder-es.png` (ES desktop, «на кого лицензия» → «существует?»),
`ca-roadmap-en.png`, `az-roadmap-ru-360.png`; запись `scroll-check.webm` (desktop RU, карточка периода — без
движения страницы).

## 11. Нерешённые вопросы (для владельца)

1. Reciprocal Classifications List CSLB по каждому штату не закодирован (UNKNOWN) — при reciprocity продукт говорит «проверьте официальную страницу»; можно добавить, если владелец даст список.
2. Arizona: всё из §5 — нужен доступ к roc.az.gov или текст страниц от владельца.
3. В плане при «лицензия выдана» карточка отпечатков показывается «Completed» (вывод из факта выдачи, не из ответа); результаты экзаменов при этом «—». Оставить вывод или тоже показывать «—» — решение владельца.
4. Миграция `licenseNumber → newLicenseNumber` намеренно не трогает неоднозначные сохранённые записи (лицензия была И выдана новая): там номер остаётся в `licenseNumber`, пользователь может ввести новый в блоке «выдана».
5. `ROADMAP_STATES.az.available = true` сохраняется с прошлого раунда — при merge страница AZ станет доступна.

## Source References

`lalianamen/llicena@ef5fd71`: `js/roadmap/app-roadmap.js`, `js/roadmap/roadmap-logic.js`, `js/roadmap/roadmap-config.js`,
`js/roadmap/i18n-roadmap.js`, `css/roadmap.css`, `roadmap.html`, `roadmap-az.html`, `scripts/test-roadmap-v3.mjs`,
`scripts/verify.js`; scratchpad-сюиты `bugs-suite.mjs`, `scroll-check.mjs`, `cv-suite*.mjs`, `exp-suite*.mjs`,
`dates-suite.mjs`, `biz-suite.mjs`, `az-suite.mjs`, `preview-check3.mjs`; страницы CSLB (§4), форма 13A-1
(rev. 01/2026); статуты azleg.gov (§5).

## Verification Status

**Partially Verified** — код и тесты проверены чтением и прогоном в песочнице; правила CSLB — чтением
названных страниц 2026-09-05; ROC-страницы недоступны (UNKNOWN отмечены явно); Supabase-путь сохранения
не прогонялся.
