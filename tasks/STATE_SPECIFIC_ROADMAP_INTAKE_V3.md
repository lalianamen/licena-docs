# TASK — State-Specific Adaptive Roadmap Intake V3

## Status

**READY_FOR_REVIEW**

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
