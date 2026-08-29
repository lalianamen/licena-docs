# REPORT — Adaptive Licensing Roadmap: аудит source state `037bdd9`

Задача: `tasks/ADAPTIVE_LICENSING_ROADMAP.md` · Dispatch: `lalianamen/LLICENA#188`
Дата: 2026-08-29 · Автор: Claude (интерактивная сессия владельца, не claude[bot])
Проверяемое состояние: private `LLICENA@main` = `037bdd9` («Adaptive Licensing Intake v2»)

## 1. Обязательное раскрытие: `037bdd9` уже на main и уже задеплоен

- Коммит `037bdd9` был смержен в private `main` и **задеплоен в production (licena.us)** этой же интерактивной сессией 2026-08-29 ≈05:15 UTC. Байты на проде сверены с исходниками по sha256 — совпадают (roadmap.html, app-roadmap.js, i18n-roadmap.js, roadmap-config.js, roadmap.css, license-roadmaps-v2.sql).
- Деплой выполнен ПО РАНЕЕ ДЕЙСТВОВАВШЕМУ owner-approved флоу, ДО публикации протокола `tasks/WORKFLOW.md` (`f32680a`, 2026-08-29 05:49 UTC). С момента `f32680a` дальнейшие merge/deploy в private `main` этой сессией не выполняются без статуса **APPROVED_FOR_MAIN**.

## 2. Аудит по критериям задачи (source state `037bdd9`)

| Критерий задачи | Статус | Как проверено |
|---|---|---|
| Adaptive journey A–L, не 4 фиксированных экрана | ✅ | 10 условных экранов, гейты по ответам; e2e-персоны A–J проходят разные наборы экранов |
| Entity: have / plan / sole owner; для существующей — type, name, status | ✅ | ветка haveEntity, entityName, entityActive; для новой — только intent + SOS note |
| DBA ≠ entity type, хранится раздельно | ✅ | отдельный вопрос, отдельные колонки `entity_type` / `dba` |
| Experience: years / type / employers / certifier / records, без eligibility-выводов | ✅ | 5 полей; статус-строка «Experience status» с осторожными формулировками |
| Application: submission date, status, corrections (+notes), accepted/returned | ✅ (частично, см. gaps) | экран app: дата подачи, 4 статуса, corrAt+corrNotes; **application number не собирается — gap G3** |
| submitted ≠ accepted (classification/experience не «одобрены» от подачи) | ✅ | derive(): completed только при st_accepted; персоны C/D это ассертят |
| Dates as first-class data (8 дат) | ⚠️ | 5 из 8 в normalized-колонках (`license-roadmaps-v2.sql`); **нет: accepted date, correction deadline, final requirement deadlines — gaps G1/G2** |
| Exams раздельно, 4 статуса | ✅ | lawStatus/tradeStatus: not/scheduled/passed/failed + даты; retake-note (21 день / 18 мес, источник Examination FAQ) |
| Current Position ≠ Next Action ≠ Parallel ≠ Next Important Date | ✅ | 4 независимых пути кода: positionOf / recommendNext / par|wait-блок / nextDate |
| Консервативный progress | ✅ | только completed-шаги по весам; issuance — только по явному подтверждению пользователя |
| Ready to Work: условный по entity, официальные источники, нейтральный финал | ✅ | FBN (BPC 17910) / CalGold / EDD / SOS SI; LLC-note; финал «You're ready for the next stage», без Phase 2 (тест ассертит отсутствие маркетинга) |
| v1-роадмапы не ломаются | ✅ | migrate() на лету, старые поля сохранены (обратимо); персона J |
| Supabase-фолбэк до применения SQL | ✅ | dbSave: расширенная строка → при ошибке v1-строка; localStorage всегда зеркалит |
| EN/RU/ES | ✅ | 325 ключей/язык, паритет проверен скриптом |
| Auth/payments/courses/analytics/routes не тронуты | ✅ | диффы только в js/roadmap/*, css/roadmap.css, 4 html (только `?v=` пины), supabase/sql/license-roadmaps-v2.sql |

Подтверждённые gaps относительно task-файла: **G1** (accepted date), **G2** (correction 90-day deadline + 18-month exam deadline не выводятся; источники: CSLB Application_Returned.aspx, Examination_FAQ.aspx), **G3** (optional application number), **G4** (time/location событий). Совпадают с discovery-отчётом claude[bot] в `LLICENA#188`; их имплементацию ведёт claude[bot] на ветке `claude/ticket-adaptive-licensing-dates` (на момент отчёта ветка на origin ещё не запушена, PR нет).

## 3. Проверки (source state `037bdd9`, все выполнены и просмотрены)

- `node scripts/verify.js` — 0 ошибок, 138 JS-файлов OK.
- Playwright e2e персоны A–J — **74 проверки, все PASS**: новичок без certifier (A), готов подавать (B), подано+ожидание+отпечатки requested (C), correction (D), accepted/экзамены не назначены (E), trade через 10 дней — countdown+guidance (F), failed Law&Business — retake (G), оба сданы — final reqs C-39 WC-always (H), issued corporation+DBA+наём — RTW 4 пункта и нейтральный финал (I), миграция legacy v1 (J); плюс demo, RU 360 без h-scroll, ветка AZ.
- Сьют Application Assistant — 43 проверки PASS (включая prefill entity из роадмапа).
- Рендеры просмотрены глазами: EN desktop, RU 360, RTW-финал.

## 4. Preview и скриншоты (точно `037bdd9`)

- Живой preview: https://licena.us/roadmap.html?demo=1 (EN; `?lang=ru&demo=1` / `?lang=es&demo=1`) — прод уже на `037bdd9`, см. раскрытие в §1.
- Скриншоты в этом репо: `tasks/reports/img/adaptive-v2-en-desktop.png`, `tasks/reports/img/adaptive-v2-ru-360.png`, `tasks/reports/img/adaptive-v2-rtw-issued.png`.

## 5. Миграции

- `supabase/sql/license-roadmaps.sql`, `license-applications.sql`, `license-roadmaps-v2.sql`: статус применения в живой Supabase — **UNKNOWN** (применяет владелец вручную в SQL Editor). Клиент до применения работает на localStorage/v1-строке; ничего не ломается. SQL не применялся из сессии и не будет применяться без решения владельца.

## 6. Официальные источники (проверены live 2026-08-29, URL с датами — в `roadmap-config.js`)

CSLB: Licensing_Classifications_Detail per-class (12 URL), форма 13A-11 (rev. 12/2024), Fingerprint_Q_And_A, CheckApplicationII, Processing Times, Bond Information, Workers' Compensation (BPC 7125, WC-always список), Examination_FAQ (21 день/18 мес), asbestos open-book (BPC 7058.5), LLC.aspx (BPC 7071.6.5/7071.19). RTW: BPC 17910 (FBN), CalGold, EDD employer registration, SOS Statements of Information, bizfileonline.sos.ca.gov.

## 7. Нерешённое / риски

- Gaps G1–G4 — в работе у claude[bot] (отдельная feature-ветка, пойдёт по циклу WORKFLOW: preview → READY_FOR_REVIEW → решение владельца; без merge в main).
- Инфраструктуры доставки напоминаний (email/push) в LICENA нет — собраны данные + `reminder_opt_in` (отдельно от маркетинговых согласий) + подсказки на дашборде; доставка честно не имитируется.
- Статус применения SQL — UNKNOWN (см. §5).

## 8. Статус

Для source state `037bdd9`: **READY_FOR_REVIEW** — доказательства полны (аудит §2, проверки §3, preview/скриншоты §4, миграции §5, источники §6). Общий статус task-файла остаётся `IN_PROGRESS` до завершения ветки G1–G4 (claude[bot]). Дальнейшие merge/deploy — только после **APPROVED_FOR_MAIN**.
