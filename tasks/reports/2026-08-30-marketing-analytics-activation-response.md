# Отчёт — ответ на задачу MARKETING_ANALYTICS_ACTIVATION

Задача: `tasks/MARKETING_ANALYTICS_ACTIVATION.md`
Дата: 2026-08-30
Связанные документы: `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`,
`tasks/reports/2026-08-30-marketing-aggregates-runbook.md`,
`tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`

## Status

**Задача выполнена не была — она устарела к моменту постановки.** Всё, что
она предписывает подготовить к активации, уже активировано владельцем в
production до её появления. Ниже — фактическое состояние по каждому её
пункту и список того, что действительно осталось незакрытым.

## Расхождение предпосылок задачи с фактами

| Утверждение задачи | Факт на 2026-08-30 | Источник |
|---|---|---|
| reviewed commit `0b5ea3a` | актуальный `ae13124`: после Review 3 добавлены `94c9157` (cron-SQL) и `ae13124` (revoke дефолтных грантов) | git-история осн. репо |
| «production Supabase does not contain the proposed marketing aggregate tables» | содержит все 4 таблицы; функция задеплоена; cron активен; выполнен backfill за 30 завершённых PT-дней | `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md` |
| «Do not merge to private `main`» | код смержен в `main` (`ae13124`) по прямому указанию владельца | указание владельца в интерактивной сессии 2026-08-30 |
| «Do not apply migrations, deploy Edge Functions, create/change cron, or write backfill rows in production without APPROVED_FOR_MAIN» | миграция, деплой, cron и backfill выполнены владельцем вручную по runbook | там же |

Правила 1 и 2 раздела «Mandatory safety rules» перекрыты прямым
распоряжением владельца, отданным до постановки этой задачи. Остальные
правила (3–7) соблюдены: деструктивные проверки на production не
выполнялись, секреты и PII в licena-docs не публиковались, `daily-stats` не
изменялся, auth/payments/courses/RLS/маршруты не затрагивались.

## Private branch and commit SHA

- Ветка разработки: `claude/marketing-analytics-aggregates` @ `ae13124`.
- `main` основного репозитория: `ae13124` (смержено 2026-08-30).
- Изменение против прежнего `main` (`037bdd9`): 6 новых файлов, 854 строки,
  **только добавления**; изменённых или удалённых файлов нет.

## Environment used for validation

- **Логика** — локально: `node --experimental-strip-types
  scripts/test-marketing-core.mjs` (48 проверок, импорт production-модуля
  `core.ts`), `node scripts/check-channel-parity.js`, `node scripts/verify.js`.
- **База данных и функция** — **production Supabase**, все шаги выполнял
  владелец через Dashboard. Изолированной branch database в проекте нет;
  у AI-сессии доступа к Supabase нет вообще (ни ключей, ни branch DB).
  Требование задачи «использовать изолированную среду» выполнено НЕ было:
  проверка на production состоялась раньше постановки задачи.

## Migration applied / not applied and where

Применена в production 2026-08-30 (`supabase/sql/marketing-daily-aggregates.sql`).
Созданы `marketing_daily_metrics`, `marketing_channel_daily`,
`marketing_page_daily`, `marketing_state_snapshots` и функция
`marketing_replace_day(date, jsonb, jsonb, jsonb)`.

## Test matrix (по пунктам Phase 1 задачи)

| № | Проверка | Результат | Доказательство |
|---|---|---|---|
| 1 | Миграция применяется на совместимой схеме | PASS | выполнена в production без ошибок |
| 2 | 4 таблицы и их семантика | PASS | схема + выгрузка `information_schema` |
| 3 | RLS включён на всех новых таблицах | PASS | 4 строки, все `RLS=true` |
| 4 | `anon` / `authenticated` не читают и не пишут | **PARTIAL** | RLS без политик + выполненный `revoke`; повторный запрос грантов ПОСЛЕ `revoke` не выполнялся → `UNKNOWN`; эмпирический негативный тест не проводился |
| 5 | День заменяет только привилегированный путь | PASS | EXECUTE на `marketing_replace_day` только у `postgres` и `service_role` |
| 6a | Два прогона за одну дату → одна строка на ключ | PASS | повторный идентичный вызов: 62/491 строк без изменений, дублей 0/0, 30 строк / 30 дней |
| 6b | Исчезнувшие ключи каналов/страниц не переживают пересчёт | PASS | покрыто core-тестом (полнота набора строк дня) + семантика `delete+insert` в RPC |
| 6c | Сбой не оставляет частично заменённый день | PASS по конструкции | замена дня — одна транзакция в `marketing_replace_day`; эмпирический тест сбоя не ставился |
| 7 | Матрица авторизации | **PARTIAL** | подтверждена только ветка «сервисные credentials → принято» (успешные вызовы шагов 5 и 6). Ветки «без ключа», «anon-ключ», «пользовательский JWT» → 403 НЕ проверены эмпирически |
| 8 | Постраничное чтение, fail-closed | PASS (юнит) | тесты `fetchAllRows`; на живых данных не срабатывало — прочитано 1970 строк при лимите 400000 |
| 9 | DST/календарь + полный прогон core-тестов | PASS | 48/48, повторный прогон 2026-08-30 |
| 10 | Санитизация path, отсутствие query-строк и PII | PASS (код+схема) | `pathOnly` обрезает всё после `?`; `check (char_length(path) between 1 and 200)`; в таблицах нет колонок user_id/device/ip/ref. Контрольный запрос на живых данных не выполнялся |
| 11 | Исключение тестеров | PASS (код) | фильтр `profiles.is_tester` во всех пользовательских метриках; на уровне устройств до входа фильтрация невозможна — ограничение задокументировано |
| 12 | Parity таксономии каналов с `daily-stats` | PASS | `check-channel-parity.js`: 23 строки идентичны |
| 13 | Database advisors | PASS | Security Advisor: 0 errors, 22 warnings, 11 suggestions; ни одного про `marketing_*` |
| 14 | Откат проверен | **FAIL — не выполнялся** | SQL отката написан (закомментированные `DROP` в миграции), но ни разу не исполнялся; на production проверять нельзя, изолированной среды нет |
| 15 | `daily-stats` без диффа и работает независимо | PASS | дифф против `main` — 0 строк; отдельная функция, отдельное расписание (15:00 против 15:30 UTC) |

## RLS / grant results

RLS включён на 4 таблицах, политик нет. EXECUTE на RPC — только `postgres`
и `service_role`. Обнаружено и исправлено: default privileges Supabase в
схеме `public` выдали `anon`/`authenticated` все привилегии на новые
таблицы, включая `TRUNCATE`, который **не подчиняется RLS**; добавлен явный
`revoke` (в миграцию — `ae13124`; на живой базе выполнен). Повторная
проверка после `revoke` — `UNKNOWN`.

## Advisor results

0 errors. 22 warnings и 11 suggestions относятся к объектам, существовавшим
до этой задачи; содержание этих 22 предупреждений в рамках данной работы не
разбиралось — `UNKNOWN`.

## Idempotency / atomicity results

Повторный идентичный вызов функции: 62 строки каналов и 491 строка страниц
без изменений, дублей по обоим составным ключам 0, строк за текущий неполный
день 0, в `marketing_daily_metrics` ровно 30 строк / 30 дней.

## Authorization results

Подтверждено: вызов с сервисными credentials принимается (формат
`sb_secret_...` проходит через проверку возможностей на
`/auth/v1/admin/users`). Негативные ветки не проверены — см. «Unresolved
blockers».

## DST and no-PII results

DST: список целевых дней строится календарной арифметикой от текущей
Pacific-даты (`core.targetDaysBack`); 11 тестов, включая оба repro-кейса
Review 2 — PASS. PII: в 4 таблицах нет колонок с email, UUID пользователя,
идентификатором устройства, IP или реферером; `path` хранится без query-строки.

## Backfill window

Выполнен: 30 завершённых Pacific-дней, 2026-07-30 … 2026-08-28; прочитано
1970 исходных строк; записано 30 дневных, 62 канальных, 491 постраничная
строка; снепшот состояния — за 2026-08-29.

## Activation and rollback summary

Активация выполнена по `tasks/reports/2026-08-30-marketing-aggregates-runbook.md`
(7 шагов: миграция → проверка RLS/грантов → advisors → деплой функции →
backfill → проверка идемпотентности → cron). Откат описан закомментированными
`DROP` в миграции и `cron.unschedule` для расписания; на практике не
проверялся.

## Unresolved blockers

1. **Негативные тесты авторизации (пункт 7).** Не проверено, что вызов без
   ключа, с публичным anon-ключом и с пользовательским JWT возвращает 403.
   Требуется действие владельца (три вызова функции) — у AI-сессии нет
   доступа к ключам.
2. **Повторная проверка грантов после `revoke` (пункт 4).** Один запрос к
   `information_schema.role_table_grants`. Требуется действие владельца.
3. **Проверка отката (пункт 14).** Требует изолированной branch database:
   на production выполнение `DROP` уничтожит накопленную историю, а
   `marketing_state_snapshots` невосстановима (наблюдательные данные).
   Решение о создании branch database — за владельцем.
4. **Проверка санитизации path на живых данных (пункт 10).** Один
   агрегатный запрос. Требуется действие владельца.
5. **Первый автоматический запуск cron.** На дату отчёта ещё не состоялся.

## Data quality follow-up (раздел задачи)

Причина преобладания `direct` и `other` установлена по коду — это форма
таксономии, а не поломка трекинга:

- `other` — бакет для ЛЮБОГО реферера, который не Facebook / Instagram /
  Telegram / TikTok (`channelOf` в `daily-stats/index.ts`, копия в
  `marketing-aggregates/core.ts`). Отдельного бакета для поисковых систем
  нет, поэтому органика Google, Bing и т. п. попадает в `other`. Данные
  согласуются с этим: `other` — 152 визита, 69 вовлечённых (45 %), 11
  регистраций из 17 за окно; `direct` — 578 визитов при 60 вовлечённых (10 %).
- `direct` — пустой `document.referrer` (`js/pageview.js:36`,
  `js/stats.js:60`): прямой ввод адреса, закладка, QR, встроенные браузеры
  приложений и любые источники, вырезающие реферер.
- Метки `?src=` работают (`flyer` 1 визит, `facebook` 2 визита), но
  практически не используются при размещении ссылок — это дисциплина
  размещения, а не дефект кода.

`social_stats` пуст по не связанной с этим слоем причине: строки туда пишет
только `daily-stats` и только для Telegram, при условии заданного секрета
`TELEGRAM_BOT_TOKEN` и прав администратора бота в канале `@licena_us`
(`supabase/functions/daily-stats/index.ts:295–307`). Facebook и Instagram в
коде не реализованы («FB/IG later when Meta API is wired»,
`supabase/sql/report-kpi.sql`). Задан ли секрет — `UNKNOWN`.

**Минимальное предложение (НЕ реализовано, требует решения владельца):**
выделить из `other` бакеты `search` (google/bing/duckduckgo/yandex/yahoo) и,
отдельно, трафик AI-ассистентов. Ограничение: блок таксономии обязан
оставаться побайтово одинаковым в `daily-stats/index.ts` и
`marketing-aggregates/core.ts` (`scripts/check-channel-parity.js`), поэтому
изменение затрагивает и ежедневное письмо владельца. Исторические строки
`marketing_channel_daily` при этом не пересчитываются автоматически —
потребуется повторный backfill, чтобы старые дни получили новые бакеты.

## Explicit production state

- Private `main`: **изменён** — `ae13124` (по указанию владельца).
- Production database: **изменена** — 4 таблицы, 1 функция, 1 cron-задание,
  30 дней агрегатов.
- Production functions / cron: **задеплоены и активны** —
  `marketing-aggregates`, расписание `30 15 * * *` (`active = true`).
- `daily-stats`: **не изменялась**, получатели письма не менялись.

## Source References

- `tasks/MARKETING_ANALYTICS_ACTIVATION.md`
- `tasks/MARKETING_ANALYTICS_DAILY_AGGREGATES.md`
- `tasks/reports/2026-08-30-marketing-aggregates-runbook.md`
- `tasks/reports/2026-08-30-marketing-aggregates-db-validation.md`
- `tasks/reviews/2026-08-30-marketing-aggregates-review-3.md`
- `LLICENA@main` (`ae13124`): `supabase/sql/marketing-daily-aggregates.sql`,
  `supabase/sql/cron-marketing-aggregates.sql`,
  `supabase/functions/marketing-aggregates/core.ts` и `index.ts`,
  `supabase/functions/daily-stats/index.ts` (только чтение),
  `js/pageview.js`, `js/stats.js`, `supabase/sql/report-kpi.sql`,
  `scripts/test-marketing-core.mjs`, `scripts/check-channel-parity.js`
- Выгрузки Supabase Dashboard, предоставленные владельцем 2026-08-30

## Verification Status

**Partially Verified.** Утверждения о коде проверены чтением названных
файлов и прогоном проверок 2026-08-30. Утверждения о живой базе опираются на
выгрузки Supabase Dashboard, предоставленные владельцем: у AI-сессии доступа
к Supabase нет. Явно помечены как непроверенные: гранты после `revoke`,
негативные ветки авторизации, откат, санитизация path на живых данных,
первый запуск cron по расписанию, наличие секрета `TELEGRAM_BOT_TOKEN`.
