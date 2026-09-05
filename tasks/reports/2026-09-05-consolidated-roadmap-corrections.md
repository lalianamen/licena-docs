# LICENA License Roadmap — Consolidated Roadmap Corrections (отчёт для владельца)

Последняя сверка: 2026-09-05
Статус: **READY_FOR_REVIEW** (исходный статус спецификации — CHANGES_REQUESTED).
Merge в `main` и production deploy НЕ выполнялись — только после явного `APPROVED_FOR_MAIN`.

## 1. Ветка и коммиты

Репозиторий `lalianamen/llicena`, ветка `claude/state-specific-intake-v3`, запушена.
База раунда — `0967abf` (Experience Builder). Три малых проверяемых коммита:

| SHA | Шаг | Что |
|---|---|---|
| `01dc5a1` | 1 | Месяц + Год двумя полями; даты без сдвига часового пояса; время/место экзаменов; заметки по заявлению; «что попросили исправить» |
| `1ba33e0` | 2 | «Для кого лицензия?» (3 варианта); DBA перенесён на карточку подготовки заявления; строка «Ваш путь: …»; одна сводка + «Все ответы»; компактные второстепенные «Нет»; палитра/шрифты/фокус |
| `755f49a` | 3 | Правила штата отделены от движка (`ROADMAP_RULES[state]`); подключена Arizona; `roadmap-az.html`; строка «штат · классификация»; перенос периодов опыта с подтверждением; перепроверка после смены классификации |

Discovery report (шаг 0, до изменений): `/tmp` scratchpad `consolidated-discovery.md`; его выводы воспроизведены в разделе 2.

## 2. Discovery (до архитектурных изменений)

- Компоненты: `roadmap.html` (единственная страница, `data-state="ca"`), `js/roadmap/roadmap-logic.js` (чистая логика, node-тестируемая), `js/roadmap/roadmap-config.js` (данные + источники с датами), `js/roadmap/app-roadmap.js` (движок беседы + план), `js/roadmap/i18n-roadmap.js` (`TRM {en,es,ru}`), `css/roadmap.css`.
- Где жили правила штата: данные — в конфиге California, но чтение — **29 прямых обращений** к `window.ROADMAP_CA_*` внутри UI-кода `app-roadmap.js`; `STEPS = window.ROADMAP_CA_STEPS` жёстко. Второй штат без правки UI был невозможен.
- Уже исправлено ранее (проверено, не переделывалось): беседа «один вопрос за раз» (`277a993`); уровень ≠ форма занятости, карточка периода в 3 группы, certifier/документы, part-time отдельно, пересечения без удвоения (`0967abf`); скролл-прыжки Δ=0.
- Невыполненное на старте: разделение движка и правил; Arizona; строка контекста штата; перенос периодов; перепроверка при смене классификации; одна сводка (dl-таблица дублировала строки); «Ваш путь»; 3-вариантный holder; DBA в первом проходе; сетка месяцев с «Готово»; `fmtDate()` без `timeZone:"UTC"` (сдвиг на день западнее Гринвича); `nextDate()` смешивал UTC-полночь и локальную; 13 объявлений шрифтов без fallback; серые заливки.
- Риски совместимости, которые пришлось соблюсти: сохранённые `holder ∈ {me_sole, existing_biz, planned_biz, undecided}` и `entity`; legacy `level ∈ {owner_builder, self_employed}`; `dba` перестаёт быть юнитом, но значение читается; `window.ROADMAP_STAGES` читает `app-application.js:729`; node-тесты читают `window.ROADMAP_CA_EDU_CREDIT/WAIVERS/RECIPROCITY/DISASSOCIATION` — имена сохранены, `ROADMAP_RULES.ca` ссылается на те же объекты.

## 3. Что сделано по разделам спецификации

| § | Требование | Реализация (файл) |
|---|---|---|
| 1 | Аудит, малые изменения, ничего не ломать | Три коммита; auth/платежи/курсы/Supabase/аналитика/маршруты не тронуты; сохранённые ответы читаются (тест `biz-suite` B1–B3) |
| 2 | Общий движок + правила штата; Arizona вторым набором; без Nevada/Texas; честный экран для отсутствующего штата; источник и дата у каждого правила | `window.ROADMAP_RULES = {ca, az, nv:{available:false}}` в `roadmap-config.js`; `app-roadmap.js` читает только `R.*` (0 обращений к `ROADMAP_CA_*`); `roadmap-az.html`; Nevada → `stateUnavailable()` (тест `az-suite` E1); все правила AZ с `verified: "2026-09-05"` |
| 3 | Штат из страницы; контекст «штат · классификация»; независимость прогресса; перенос периодов с подтверждением; перепроверка зависимых ответов | `.cv-ctx` над анкетой и `stateName` в шапке плана; ключи `lp:roadmap:<state>[:<trade>]` и `dbLoad().eq("state", STATE)` (тесты B7, C1, C7); `foreignPeriods()` + предложение «Скопировать периоды / Нет» — копируются только факты (компания, даты, работа, часы), уровень/подтверждение/документы спрашиваются заново, статусы не переносятся (C3–C8); `recheckAfter("class")` открывает блоки «заявление» и «waiver» с пометкой, не удаляя ответы (D1–D3) |
| 4 | Без фиксированной последовательности; убрать дубли; одна сводка + «Все ответы»; «Ваш путь: …»; классификация обязательна | `summaryEl(all, doneN)`: строка-итог + `<details class="cv-all">` со строками и «Изменить»; dl-таблица удалена; `.cv-path` со строкой «Ваш путь: …» и раскрытием «Что это значит» (предварительная маршрутизация); второстепенные «Нет» — `.cv-rowmin`; число в `expCaution` больше не повторяет сетку |
| 5 | «Для кого лицензия» (я / существующая / планируемая) с пояснением; existing → тип, название, статус; planned → намерение; DBA — атрибут, в подготовке заявления; tribal не в основных; без универсального вопроса о квалифицирующем лице | Юнит `holder` переписан; `qualifySelf` больше не спрашивается (уточнение для владельца с другим qualifying party — блок «цель», `when: rmeOwner`); DBA — три кнопки на карточке `prepare_application` (`RM.answers.dba`, питает RTW FBN); tribal — за «Другой тип организации» только в CA (`entityUI.other`) |
| 6–7 | Experience Builder, verification | Сделано в `0967abf`; в этом раунде — только уровни/форма/подсказки per state |
| 8 | Месяц + Год без «Готово»; «работаю до сих пор»; проверки; точные даты с временем/местом; UTC/DST/конец месяца | `monthYear()` — два `<select>`; `isPartial()` → «Укажите и месяц, и год»; `fmtDate()` через `RML.parseDate` + `timeZone:"UTC"` + год; `todayIso()` — локальный календарный день; `nextDate()` на `RML.daysBetween`; поля `lawWhere`/`tradeWhere` (колонки `law_exam_where`/`trade_exam_where` из `license-roadmaps-v3.sql`), `appNotes`, `corrWhat` (в `answers jsonb`); тест `dates-suite` в America/Los_Angeles и Pacific/Kiritimati |
| 9 | Палитра, поля, hover, фокус, шрифты, заливки | `font-family:'Archivo',sans-serif` / `'IBM Plex Mono',monospace` везде; hover `#EEF2F7`; фокус `outline:3px solid var(--ink)`; `.rm-ready`, `.rm-ent-more` — белые; сетка месяцев и `.cv-res` удалены |
| 10 | Скролл-прыжки | Причина и фикс — в отчёте `2026-09-05-experience-builder-rework.md`; перезаписана запись на итоговом SHA (`img/2026-09-05-consolidated-scroll-check.webm`); `exp-suite2` (68) измеряет Δ по каждому ответу |
| 11 | Полный путь после подачи | Сохранён; добавлены заметки, «что попросили исправить», время/место экзаменов и Live Scan на карточках; для AZ — без Application Assistant (`R.assistant=false`, честная заметка `rmAppNone`) |
| 12 | Проверка, малые коммиты, превью с того же SHA, отчёт | Ниже |

## 4. Arizona — что взято из официальных источников (azleg.gov, прочитано 2026-09-05)

| Источник | Факт в продукте |
|---|---|
| A.R.S. § 32-1122(E)(1) | 4 года practical/management опыта, из них ≥2 за последние 10 лет; техническое обучение ≤2 лет; registrar «may reduce»; документация/проверка опыта «shall waive» для текущего/прежнего qualifying party той же классификации |
| § 32-1122(E)(2), (F) | письменный экзамен «not more than two years before application, if required»; «shall waive the examination requirement» для qualifying party той же классификации за 5 лет — в продукте только «может быть освобождён — ROC подтверждает по записям» |
| § 32-1122(B), (G), (H) | содержание заявления (классификация, лица компании, qualifying party, good standing в Corporation Commission, workers' comp attestation); несовершеннолетним не выдаётся; отпечатки «may require» + сборы § 41-1750 |
| § 32-1124(A), (B), (D) | уведомление в течение 60 дней после полного заявления; номер «ROC» на объектах, сметах, рекламе (пункт Ready to Work); сбор отклонённого заявления не возвращается |
| § 32-1127.01 | 15 дней уведомить, 60 дней переквалифицироваться, иначе suspension by operation of law (срок `dateKindRme` для AZ) |
| § 32-1152 | surety bond / cash deposit до выдачи, размеры по классификации и объёму; residential/dual — recovery fund или +$200,000 |
| azcc.gov | Arizona Corporation Commission — источник для good standing (страница жива 2026-09-05) |

**UNKNOWN (roc.az.gov и apps.azsos.gov отвечают HTTP 403):** вендор и точные названия экзаменов, перечень классификаций по правилу, размеры сборов, форма подтверждения опыта и кто её подписывает, сроки обработки, взаимность. В конфиге эти ключи `null`/отсутствуют; в UI такие вопросы/заметки скрыты, а не заменены калифорнийскими.

## 5. Изменённые файлы

`js/roadmap/roadmap-logic.js` (v5), `js/roadmap/roadmap-config.js` (v10), `js/roadmap/app-roadmap.js` (v16), `js/roadmap/i18n-roadmap.js` (v15; базовые ключи 688 × 3 языка + слой `_az` 135 ключей × 3), `css/roadmap.css` (v14), `roadmap.html`, **новый** `roadmap-az.html`, `js/app-cabinet.js` (href кнопки «Open my roadmap» по выбранному штату), `scripts/test-roadmap-v3.mjs` (+24 проверки). SQL/Edge Functions/платежи — без изменений.

## 6. Тесты, typecheck, сборка

Проект — vanilla HTML/CSS/JS без сборки и без TypeScript: «typecheck/build» здесь = `node --check` всех JS через `node scripts/verify.js` (139 файлов OK, банки/офферы синхронны).

| Проверка | Результат на `755f49a` |
|---|---|
| `node scripts/test-roadmap-v3.mjs` | 138 PASS (календарь/DST/високосный год; уровни; окно AZ 4+2; CA-клиппинг; waivers CA/AZ; импортированный период; правила штатов) |
| Playwright `dates-suite` | 34 PASS — поля Месяц+Год, неполная дата, окончание раньше начала, будущий месяц, «работаю до сих пор», восстановление после reload; точные даты/время/место/заметки без сдвига в America/Los_Angeles и Pacific/Kiritimati |
| Playwright `biz-suite` | 40 PASS — holder-поток, DBA на карточке, «Ваш путь», сводка + «Все ответы», совместимость старых `holder/undecided/dba`, RU 360, fallback шрифтов, hover, фокус |
| Playwright `az-suite` | 50 PASS — страница AZ без CSLB/California, независимость CA/AZ, перенос периодов, смена классификации, экран Nevada, RU 360 |
| Регрессия `cv-suite` / `cv-suite2` / `exp-suite` / `exp-suite2` | 52 / 48 / 39 / 68 PASS (сюиты обновлены под новый контрол дат и отсутствие DBA в первом проходе) |
| Консольные ошибки | 0 (внешние шрифты блокирует прокси песочницы — отфильтровано как не-приложение) |

Недоступные/неполные проверки (честно): нет unit-фреймворка и typecheck (природа проекта); `ffmpeg` в песочнице отсутствует — кадры записи не извлекались, запись проверена только по размеру/длительности прогона; Supabase-сохранение не прогонялось (нет сессии в песочнице) — проверялся localStorage-путь; страницы ROC не читались (403).

## 7. Миграции и rollback

Новых SQL нет. Новые ответы (`appNotes`, `corrWhat`, `priorQP`, `priorQP5y`, `recheck`, `expImportDismissed`, `imported` у периода) живут в `license_roadmaps.answers jsonb`; `lawWhere`/`tradeWhere` используют колонки, добавленные ранее в `supabase/sql/license-roadmaps-v3.sql` (статус применения — UNKNOWN, клиент откатывается к короткой строке при ошибке). Rollback = revert трёх коммитов; данные не теряются.

## 8. Превью

https://claude.ai/code/artifact/5c1a7ef2-4ed3-420a-af44-342ff6a3602a — собрано из тех же файлов, что и `755f49a` (self-contained, без запросов к сети). Селектор сверху: «0 · Анкета (California)», «AZ · Анкета (Arizona)», сценарии 1–11. Проверено Playwright (`preview-check3.mjs`): CA/AZ анкеты, сценарий `exams`, ширина 360 — без ошибок консоли.

## 9. Скриншоты и запись

`tasks/reports/img/2026-09-05-consolidated-*`: `ca-holder-en.png`, `ca-summary-en.png`, `ca-roadmap-en.png`, `ca-dba-card-en.png`, `ca-exp-ru-360.png`, `az-exp-en.png`, `az-roadmap-en.png`, `az-intake-ru-360.png`; запись скролл-проверки `scroll-check.webm` (desktop RU, карточка периода: все ответы без движения страницы).

## 10. Нерешённые вопросы (для владельца)

1. Arizona: всё, что живёт только на roc.az.gov (см. §4 UNKNOWN), — нужен доступ к ROC-страницам или их текст от владельца, чтобы добавить названия экзаменов, сборы, форму подтверждения опыта, взаимность.
2. Уровни опыта AZ названы словами статута («practical» / «management»); формулировка ROC-формы не проверена.
3. Ready to Work для AZ содержит только статутный пункт (номер ROC); торговое имя/локальные лицензии не проверены и не добавлены.
4. Application Assistant для AZ отсутствует — на карточке честная заметка.
5. Владелец решает: `ROADMAP_STATES.az.available` теперь `true` — при merge Arizona-страница станет доступной по прямой ссылке и из кабинета.

## Source References

`lalianamen/llicena@755f49a`: `js/roadmap/app-roadmap.js`, `js/roadmap/roadmap-logic.js`, `js/roadmap/roadmap-config.js`, `js/roadmap/i18n-roadmap.js`, `css/roadmap.css`, `roadmap.html`, `roadmap-az.html`, `js/app-cabinet.js`, `scripts/test-roadmap-v3.mjs`, `scripts/verify.js`, `supabase/sql/license-roadmaps-v3.sql`; scratchpad-сюиты `dates-suite.mjs`, `biz-suite.mjs`, `az-suite.mjs`, `cv-suite*.mjs`, `exp-suite*.mjs`, `preview-check3.mjs`; статуты azleg.gov (§4).

## Verification Status

**Partially Verified** — код и тесты проверены чтением и прогоном в песочнице; факты Arizona — чтением статутов на azleg.gov 2026-09-05; ROC-страницы недоступны (UNKNOWN отмечены явно); Supabase-путь сохранения не прогонялся.
