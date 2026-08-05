# 26 — CHANGELOG

Последняя сверка: 2026-08-05
Append-only: записи не удаляются и не переписываются (`CLAUDE.md` §3);
ошибочная запись отменяется новой записью. Две части: история продукта LICENA
(только события, подтверждённые файлами/git основного репозитория) и журнал
этой базы знаний.

## Часть 1 — история продукта LICENA (задокументированная)

История до 2026-06-22 — UNKNOWN: клон основного репозитория shallow (видны
50 коммитов, 2026-07-28…2026-08-04); более ранние даты ниже взяты из
датированных файлов и комментариев с пометкой «owner decision/request/spec».

| Дата | Событие | Источник |
|---|---|---|
| 2026-06-22 | Миграция `user_courses`: колонка `status` (active/inactive), политики update/delete | `docs/migration-2026-06-22-course-status.sql` |
| 2026-07-14 | Решение владельца: цена подписки $20/месяц за курс | комментарий `PRICE_CENTS` в `supabase/functions/stripe-checkout/index.ts` |
| 2026-07-16 | Решение владельца: бесплатные сертификационные банки right-sized (epa-608 4×50, osha 5×50, asbestos 5×20), а не 500 | `CLAUDE.md` осн. репо |
| 2026-07-17 | First-party аналитика `page_views` + ежедневный отчёт daily-stats | `supabase/sql/page-views.sql`, `supabase/functions/daily-stats/index.ts` (owner request) |
| 2026-07-18 | Каркас подписок: started_at/expires_at/auto_renew/stripe_subscription_id, `course_trials`, cron `expire-subscriptions` | `supabase/sql/subscriptions-schema.sql` (owner spec) |
| 2026-07-19 | Гайд `contractor-business` выдан всем аккаунтам (и добавляется новым автоматически) | `supabase/sql/backfill-business-guide.sql` |
| 2026-07-20 | Лимит устройств повышен с 3 до 5; бесплатные курсы сгруппированы в подгруппу «Free Courses» | `supabase/devices_anti_sharing.sql`; `js/catalog/ca.js` (owner decisions) |
| 2026-07-21 | Новый платный банк C-27 Landscaping: 500×3 языка | `js/bank-updates.js` |
| 2026-07-28 | Stripe-аккаунт активен (go/no-go); новый банк B-2 (500×3); аудиты банков C-8 и C-27 по официальным источникам; тестерам продлён платный доступ до 31 октября | `supabase/sql/stripe-payments.sql`; `js/bank-updates.js`; git `main` (#138); `supabase/sql/extend-tester-trials-oct31.sql` |
| 2026-07-29 | Платные банки удалены из `main` (статика скачивалась свободно); исходники — ветка `content-banks-src`, сайт отдаёт их из Supabase `bank_questions` (RLS) | `CLAUDE.md` осн. репо; git `main` (#139) |
| 2026-07-31 → 2026-08-01 | Конец бесплатной беты; включён платный режим: RLS-политики free-tier-only, разовый 3-дневный триал без карты | `supabase/sql/restore-beta-policies-until-aug1.sql`, `stripe-payments.sql` SECTION B, `trial-3day.sql` (owner decision 2026-08-01) |
| 2026-08-01 | Тестерские аккаунты: без лимита устройств; механизм staged-курсов `test:true` | `supabase/devices_anti_sharing.sql`, `supabase/sql/tester-account.sql` (owner decisions) |
| 2026-08-03 | Новый платный банк C-33 Painting: 500×3 | `js/bank-updates.js` |
| 2026-08-04 | Запуск Arizona: банк AZ SRE (500×3, первый не-калифорнийский курс), AZ-гайды + CA↔AZ reciprocity, honest-chances для SRE; первый SEO-проход по данным GSC; «free sample»-фрейминг practice-страниц | `js/bank-updates.js`; git `main` (#182–#187) |

Подготовлено, не выпущено (на 2026-08-05): банк `la-business-law` (500 EN +
RU/ES) на ветке `content-banks-src`; армянский язык C-20 в staged-режиме
(`c20-exam.hy.js`, `testLangs`).

## Часть 2 — журнал базы знаний licena-docs

| Дата | Изменение | Коммит |
|---|---|---|
| 2026-08-05 | `CLAUDE.md` заполнен постоянной инструкцией (роль, правдивость, структура 01–30, синхронизация, публичность) | `ee269c0` (merge в main: `e2936eb`, PR #1) |
| 2026-08-05 | Этап 1: созданы `README.md`, `01_PROJECT_OVERVIEW.md`, `02_ARCHITECTURE.md`, `03_REPO_STRUCTURE.md`, `28_AI_CONTEXT.md` | `b63275d` (merge в main: `f974ada`) |
| 2026-08-05 | Аудит-стандарт (`CLAUDE.md` §9): разделы Source References + Verification Status во всех документах; созданы `audit/INDEX.md`, `audit/COVERAGE.md` | `36fea4c` (merge в main: `9a78dd0`) |
| 2026-08-05 | Итерация «архитектура»: созданы `04_DATABASE.md`, `05_API.md`, `13_UX.md` (Features), `26_CHANGELOG.md`; обновлены `01`–`03`, `README.md`, `audit/*` (верификация повышена: все SQL-файлы и три Stripe-функции прочитаны полностью) | см. текущий коммит в git-истории |

## Source References

- Часть 1: файлы, названные в колонке «Источник» (все прочитаны 2026-08-05);
  git-история `main` основного репо (shallow, 50 коммитов).
- Часть 2: git-история репозитория licena-docs.

## Verification Status

**Partially Verified** — события Части 1 подтверждены названными файлами;
полнота истории не гарантирована (shallow-клон, UNKNOWN до 2026-06-22);
даты «owner decision» взяты из комментариев в коде, а не из внешних записей.
