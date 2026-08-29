# TASK — Adaptive Licensing Roadmap / Intake

Последняя сверка: 2026-08-29

## Status

**IN_PROGRESS**

Private dispatch: `lalianamen/LLICENA#188`

2026-08-29: аудит source state `037bdd9` завершён — **READY_FOR_REVIEW** для этого
состояния (см. `tasks/reports/2026-08-29-adaptive-roadmap-audit-037bdd9.md`:
раскрытие о деплое, критерии, проверки, preview, скриншоты, миграции — UNKNOWN).
Общий статус остаётся IN_PROGRESS: gaps G1–G4 (даты) ведёт claude[bot] на
отдельной feature-ветке.

## Objective

Провести аудит текущей реализации roadmap/intake в приватном репозитории `lalianamen/LLICENA`, сравнить её с требованиями ниже и затем реализовать adaptive licensing intake небольшими обратно совместимыми изменениями.

Не переписывать систему целиком до завершения discovery. Не merge/deploy в `main` без статуса **APPROVED_FOR_MAIN**.

## Product scope

LICENA — мультиязычная платформа contractor licensing и exam preparation:

- языки: English, Russian, Spanish;
- текущие штаты: California и Arizona;
- текущий основной продукт: подготовка к contractor licensing exams;
- направление: путь от «I want a contractor license» до выдачи лицензии и короткого этапа «Ready to Work».

## Mandatory safety rules

1. Не редизайнить весь сайт.
2. Не ломать auth, payments, существующие courses, Supabase, analytics и production routes.
3. До архитектурных изменений изучить текущий private repository.
4. Предпочитать backward-compatible изменения.
5. Не публиковать private source code, secrets, tokens, service-role keys, PII и внутренние чувствительные детали.
6. California licensing logic проверять по актуальным официальным CSLB/government sources до кодирования.
7. Не придумывать юридические требования и не делать legal eligibility determinations.
8. DBA не является entity type.
9. `Application submitted` не означает, что classification или experience одобрены.
10. Не утверждать, что LICENA напрямую отправляет данные в CSLB, если официальной поддерживаемой интеграции нет.
11. Не merge/deploy в private `main` без явного **APPROVED_FOR_MAIN**.
12. Для UI создать preview и screenshots до запроса approval.
13. Не собирать SSN и другие highly sensitive fields без отдельного security/storage design review.

## Phase 1 — discovery first

Сначала подготовить discovery report, основанный на фактическом текущем состоянии private repository:

- текущие routes/pages;
- files/components/modules;
- текущая data model и Supabase persistence;
- roadmap derivation/progress logic;
- questionnaire flow и branching;
- i18n;
- course/classification connection;
- analytics/auth dependencies;
- migrations;
- gaps относительно этой задачи;
- regression/security/legal-content risks;
- рекомендуемый пошаговый implementation plan.

Не начинать крупный rewrite до публикации discovery. Если текущая реализация уже покрывает часть требований, сохранить её и исправлять только подтверждённые gaps.

## Required adaptive journey

Questionnaire должен отражать реальный путь и адаптироваться по ответам:

A. State  
B. License classification  
C. Business / entity structure  
D. Qualifying experience  
E. Application preparation / submission  
F. Application review / corrections  
G. Fingerprinting  
H. Law & Business exam  
I. Trade exam  
J. Final issuance requirements  
K. License issued  
L. Ready to Work

Не использовать фиксированную упрощённую анкету из четырёх экранов.

## Business / entity logic

Спросить, пользователь:

- уже имеет business/entity;
- планирует создать entity;
- будет работать как Sole Owner.

Для существующей entity собрать:

- entity type;
- legal name;
- registration status;
- DBA / fictitious business name, если применимо.

Для новой entity сохранить только intent. Полный company formation сейчас не строить.

Entity type и DBA хранить раздельно.

## Experience

Собрать достаточно данных для полезного roadmap, не определяя юридическую eligibility:

- approximate years;
- type of experience;
- employee / self-employed / owner / supervisor / combination;
- one or multiple employers;
- наличие certifier;
- наличие supporting records.

## Application

Если заявление не подано — дать нейтральную точку подключения к будущему LICENA Application Assistant.

Если подано, собрать:

- submission date;
- optional application number;
- current status;
- corrections requested;
- accepted / returned;
- notes.

Не выводить approval classification/experience только из submission.

## Dates as first-class data

Поддержать отдельные даты:

- application submitted;
- application accepted;
- correction deadline;
- fingerprint appointment;
- Law & Business exam;
- Trade exam;
- final requirement deadlines;
- license issue date.

Где уместно для события хранить date, time и location.

## Exams and courses

Law & Business и Trade exam отслеживать раздельно.

Допустимые statuses:

- not scheduled;
- scheduled;
- passed;
- failed.

Подключить exam prep к существующим LICENA courses по classification, не ломая текущие course routes или entitlement logic.

## Roadmap presentation and derivation

Явно разделить:

- Current Position;
- Your Next Action;
- Parallel Action / While You Wait;
- Next Important Date.

Не выводить завершение одного этапа из несвязанного milestone.

Progress должен быть консервативным и основываться на user-confirmed или verified completed steps, а не предположениях.

## Ready to Work

После issuance показывать только релевантные шаги, необходимые перед фактическим началом работы.

Логика должна различаться для:

- Sole Owner;
- Corporation;
- LLC;
- других реально поддерживаемых entity types.

Каждый requirement должен иметь актуальный официальный state/local source. Не превращать раздел в generic startup checklist.

После завершения Ready to Work создать только нейтральный переход к будущим business services.

## Explicitly out of scope

Не строить Phase 2:

- company setup;
- website;
- advertising;
- marketing;
- lead generation;
- branding;
- Google Business Profile;
- другие growth services.

Не реализовывать полный Application Assistant в этой задаче. Будущее направление Assistant: official field label, plain-language explanation, why CSLB asks, example, common mistakes, official source.

## Implementation sequence

После discovery:

1. Зафиксировать минимальные gaps и regression boundaries.
2. Создать отдельную feature branch от актуального private `main`.
3. Реализовать изменения небольшими логическими commits.
4. Делать schema changes только через idempotent/backward-compatible migration с rollback notes.
5. Сохранить совместимость с существующими saved roadmaps; добавить безопасную migration/normalization strategy при изменении answer shape.
6. Обновить EN/RU/ES.
7. Проверить mobile/desktop и keyboard-accessible flow.
8. Выполнить доступные tests, syntax checks, typecheck и build/verification scripts.
9. Создать non-production preview и screenshots.
10. Опубликовать безопасный итоговый отчёт в этом репозитории.
11. Установить статус **READY_FOR_REVIEW** и остановиться.
12. Не merge/deploy в private `main`.

## Required handoff report

Публиковать только:

- task status;
- implementation summary;
- private branch name и commit SHA;
- changed areas без копирования private source;
- tests/typecheck/build results;
- migrations и их application status;
- preview URL;
- screenshots для UI;
- unresolved issues;
- official sources, использованные для California logic.

Нельзя публиковать private code, secrets, PII или чувствительные implementation details.

## Status transitions

`READY_FOR_CLAUDE` → `IN_PROGRESS` → `READY_FOR_REVIEW`

При замечаниях: `CHANGES_REQUESTED`.

Merge разрешён только после явного `APPROVED_FOR_MAIN`.

Дополнительные терминальные статусы: `DO_NOT_MERGE`, `DONE`.

## Acceptance criteria

- Discovery report опубликован до крупного изменения архитектуры.
- Questionnaire адаптивно покрывает применимые sections A–L.
- DBA отделён от entity type.
- Submission не помечает classification/experience approved.
- Experience data полезны для roadmap, но eligibility не заявляется.
- Application status/corrections и важные даты представлены отдельно.
- Exams независимы и используют четыре заданных statuses.
- Roadmap различает position, next action, parallel action и next date.
- Progress консервативен.
- Ready to Work зависит от entity и подтверждён официальными sources.
- Existing auth/payments/courses/Supabase/analytics/routes не сломаны.
- EN/RU/ES согласованы.
- Tests/checks/build завершены и результаты опубликованы.
- UI preview и screenshots доступны.
- Private `main` не изменён.
- Финальный статус: **READY_FOR_REVIEW**.

## Source References

- Сообщение владельца проекта от 2026-08-29 с требованиями к adaptive licensing roadmap/intake.
- `CLAUDE.md` в `lalianamen/licena-docs`.
- Private repository `lalianamen/LLICENA` — Claude обязан указать конкретные проверенные paths/commits в discovery и handoff без копирования приватного кода.

## Verification Status

**Partially Verified** — требования подтверждены сообщением владельца; фактическое состояние private implementation должен проверить Claude внутри `lalianamen/LLICENA`.
