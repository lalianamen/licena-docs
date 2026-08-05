# 17 — Технический долг

Последняя сверка: 2026-08-05
Правило документа: технический долг = только то, что прямо подтверждается
кодом, внутренними документами, незавершённой реализацией или проверяемым
противоречием файлов. Раздел «Аудиторские гипотезы» отделяет непроверенные
предположения. Оценок и предложений исправлений нет.

Grep по `TODO|FIXME` в `js/`, `scripts/`, `supabase/`, `sw.js` — **0
совпадений**: маркеров незавершённости в коде нет. Долг проекта живёт в
устаревших описаниях и ручных синхронизациях, перечисленных ниже.

## 1. Подтверждённый долг: ручные синхронизации (задокументированы кодом)

| Что | Подтверждение |
|---|---|
| Список платных курсов дублируется в **7 местах** (2 клиентских, Edge Function, 4 SQL-списка) и синхронизируется вручную | шапка `scripts/check-paid-sync.js` — с задокументированным продакшен-инцидентом 2026-08-04: два новых банка обновили только клиентские списки → checkout падал, триалы возвращали `not_paid_course`, RLS-политика позволяла самовыдачу новых банков бесплатно. С тех пор рассинхрон — ERROR в `verify.js` |
| Лимит устройств продублирован: сервер `lim := 5` и UI-константа `MAX_DEVICES = 5` | комментарий `js/i18n-app.js:2–4` «copy and code must agree» |
| Списки в `stripe-checkout PAID` / `PAID_SUBS` / `PAID_COURSES` «keep all three in sync when a bank ships» | комментарий `supabase/functions/stripe-checkout/index.ts:23–25` |
| Статический `<head>` `index.html` и копия в `js/seo.js` требуют ручной синхронизации при смене тайтлов | спека в `docs/marketing/gsc-readout-2026-08.md` («static-baseline sync») |

## 2. Подтверждённый долг: устаревшие описания (проверяемые противоречия)

| Документ/строка | Противоречие с фактом |
|---|---|
| `README.md` осн. репо | описывает добета-состояние: «payments (Stripe) … still to come», «Free during the beta», язык `vi`, структура без `js/catalog/`, Arizona и платных банков (факты: `09_PAYMENTS.md`, `01_PROJECT_OVERVIEW.md`) |
| `CLAUDE.md` осн. репо | «the 12 paid banks» (запись 2026-07-29) — платных курсов 14 с 2026-08-04 |
| `scripts/generate-bank-csv.js:1` | комментарий «all nine PAID banks» — в списке файла 12 (на `content-banks-src` — 13) |
| `docs/schema.sql:13` | комментарий «Authorized devices per account (max 3)» — действующий лимит 5 (`devices_anti_sharing.sql`, owner decision 2026-07-20) |
| `.claude/agents/marketing.md`, блок «Current state» | «currently `?v=3`» (фактически `seo.js?v=14`), «The public site is flat (one indexable page)» (фактически 120 статических страниц + sitemap 126); staleness блока флагует и сам `gsc-readout-2026-08.md` («state block says v2 — stale») |
| `supabase/sql/*` | двойное определение `course_trials` в `subscriptions-schema.sql` и `trial-3day.sql` с разными наборами колонок (`04_DATABASE.md`); фактическая живая схема UNKNOWN |
| `js/paths.js`, шапка | ids проверяются «by hand» — «checked by scripts/verify.js's node --check only — keep ids in sync by hand when the catalog changes» |

## 3. Подтверждённый долг: незавершённое / открытое

- Открытые пункты `docs/security-todo.md` (2026-06-24), не закрытые кодом на
  2026-08-05: серверная парольная политика (п.5, серверная часть — UNKNOWN),
  валидация `courseId` при загрузке статических банков (п.4 — текущее
  состояние не перепроверялось), удаление тестового пользователя (п.9 —
  UNKNOWN). Полная таблица статусов — `08_SECURITY.md`.
- Очередь контент-аудита `docs/content-audit/queue.json`: done только
  `dmv-car` (2026-06-25), остальные позиции pending; сам файл фиксирует, что
  «in-session cron auto-expires after 7 days» и ежегодную ре-верификацию надо
  перезапускать вручную.
- Публичный чат-эндпоинт не rate-limited — зафиксировано комментарием в
  `ticket-issue/index.ts` как причина ручного greenlight (`08_SECURITY.md`).
- CSP-мета отсутствует на `privacy.html` и `terms.html` (единственные 2
  страницы из 122 HTML; grep 2026-08-05).
- `la-business-law` готов на `content-banks-src`, но не подключён (каталог,
  Stripe, trial-список) — незавершённый запуск.
- CSLB-инцидент процессов: банки c33 (2026-08-03) и az-sre (2026-08-04)
  вышли без справочных панелей — закрыто добавлением `check-course-ref.js`
  (шапка файла) и панелей (git #182).

## 4. Не долг (осознанные решения, зафиксированные в репо)

- Бесплатные банки — публичная статика: lead magnet (`CLAUDE.md` осн. репо).
- `robots.txt`-щиты — только от краулинга, не от скачивания (комментарий в
  самом файле).
- `profiles.is_tester` самоназначаем — «curtain, not a lock»
  (`tester-account.sql`).
- Отсутствие сборки/минификации собственного кода — vanilla-архитектура
  (`README.md` осн. репо).

## 5. Аудиторские гипотезы (НЕ подтверждены — требуют проверки владельцем)

- Применены ли к живой БД `stripe-payments.sql` SECTION B и весь набор
  SQL-скриптов; фактическая схема `course_trials`.
- Обновлены ли `docs/content/*-blueprint.md` после выпуска банков: шапки
  говорят «STEP 1 ONLY. No questions authored yet» (b-general-building,
  2026-07-16) и «DRAFT, pre-authoring» (c27, 2026-07-21), банки давно
  выпущены; полные тексты файлов не читались — возможно, статусы внутри
  обновлялись ниже шапки.
- Выполнена ли REVIEW-правка трёх «bare gratis»-тайтлов из
  `gsc-readout-2026-08.md` (es/c-10, es/c-39, ru/c-36).
- Язык email-шаблонов Supabase Auth (файлы не читались).

## Source References

- `scripts/check-paid-sync.js`, `check-course-ref.js` (шапки с инцидентами),
  `scripts/generate-bank-csv.js:1`, `js/i18n-app.js:2–4`, `js/paths.js`
- `README.md`, `CLAUDE.md`, `docs/schema.sql:13`, `docs/security-todo.md`,
  `docs/content-audit/queue.json`, `docs/marketing/gsc-readout-2026-08.md`,
  `.claude/agents/marketing.md` (блок Current state)
- `supabase/functions/stripe-checkout/index.ts:23–25`,
  `ticket-issue/index.ts`, `supabase/sql/subscriptions-schema.sql`,
  `trial-3day.sql`
- grep `TODO|FIXME` (0), grep CSP по всем HTML (2026-08-05)

## Verification Status

**Partially Verified.**

- Разделы 1–4 — каждый пункт подтверждён названным файлом (Verified-уровень).
- Раздел 5 — явно помеченные непроверенные гипотезы.
