# COVERAGE — карта покрытия документацией

Последняя сверка: 2026-08-05
Показывает, какие части основного репозитория `lalianamen/llicena` уже
задокументированы в базе знаний, а какие нет. Обновляется при каждом изменении
документов (`CLAUDE.md` §9).

Статусы: **документировано** (область раскрыта по существу) · **частично**
(зафиксированы состав/роли, но не содержание) · **не документировано**.

| Область основного репо | Источник (пути) | Где документировано | Статус | Что осталось |
|---|---|---|---|---|
| Страницы приложения (лендинг, кабинет, плеер) | `index.html`, `app.html`, `course.html` | 02 (роли, порядок скриптов), 03 | частично | потоки UX по шагам → `13_UX.md` |
| Публичные SEO-страницы | `practice/*`, `guides/*`, `es/*`, `ru/*`, `about/` | 02, 03 (состав, механика sample-квизов) | частично | контент и SEO-структура → `12_SEO.md` |
| Клиентские JS-модули (24 шт.) | `js/*.js` | 02 (таблица ролей), 03 | частично | построчная логика ключевых модулей не документирована |
| Каталог курсов и штаты | `js/catalog.js`, `js/catalog/*` | 01 (полный состав), 02, 28 | документировано | — |
| Банки вопросов: объёмы и форматы | `js/questions/*` (main + `content-banks-src`) | 01 (пересчёт), 03, 28 (инварианты) | документировано | содержание/методология → `10_CONTENT_MODEL.md` |
| SOP генерации контента | `docs/content/bank-playbook.md`, `*-blueprint.md`, `*-ledger.md` | 03 (существование и роль) | частично | сам SOP не читался → `10_CONTENT_MODEL.md` |
| Контент-аудит банков | `docs/content-audit/queue.json` | 02, 03, 28 (механика, политика) | частично | текущие статусы очереди не зафиксированы |
| Локализация | `js/i18n.js`, `js/i18n-app.js`, зеркала `es/`/`ru/`, `hy`-стейджинг | 01, 02 | частично | полная модель i18n → `11_I18N.md` |
| CSS / дизайн-система | `css/*.css` | 02, 03 (роли, объёмы) | частично | токены и компоненты не документированы |
| PWA | `manifest.json`, `sw.js`, `js/pwa.js` | 02 | документировано | — |
| SEO-инфраструктура | `js/seo.js`, `sitemap.xml`, `robots.txt`, `googlec…html` | 02 | частично | per-page правила seo.js → `12_SEO.md` |
| Схема БД и миграции | `docs/schema.sql`, `supabase/sql/*.sql`, `docs/migration-*.sql` | 02 (список таблиц/RPC), 03 (роли файлов) | частично | колонки, RLS-политики, миграции → `04_DATABASE.md` |
| Edge Functions | `supabase/functions/*` | 02, 03 (роли; читались только `stripe-checkout`, `assistant` частично) | частично | входы/выходы/секреты каждой → `06_FUNCTIONS.md` |
| Платежи Stripe | `stripe-checkout/-portal/-webhook`, `supabase/sql/stripe-payments.sql`, `subscriptions-schema.sql`, `trial-3day.sql` | 01, 02, 28 (цена, карта курсов, триал) | частично | webhook и SQL-слой не читались → `09_PAYMENTS.md` |
| Auth и анти-шеринг | `js/app.js`, `js/devices.js`, `docs/access-control.md`, `supabase/devices_anti_sharing.sql` | 02, 28 (модель 5 устройств/30 дней, implicit flow) | частично | полная модель доступа → `07_AUTH_ACCESS.md` |
| Безопасность | `docs/security-todo.md`, RLS, `.gitignore` | 03 (существование файла) | не документировано | → `08_SECURITY.md` |
| Аналитика first-party | `js/stats.js`, `js/pageview.js`, `supabase/sql/page-views.sql`, `report-kpi.sql`, `cron-daily-stats.sql`, `functions/daily-stats` | 02 (механика) | частично | схема отчётности → `14_ANALYTICS.md` |
| Метрики (реальные числа) | `docs/marketing/gsc-readout-2026-08.md`, `page_views`, `social_stats` | — (существование отмечено в 01, 28) | не документировано | → `15_METRICS.md` |
| Саппорт (AI + тикеты) | `js/support.js`, `functions/assistant`, `ticket-*`, `supabase/support_*.sql`, `ticket_*_trigger.sql` | 02 (роли, модель `claude-sonnet-4-6`) | частично | пайплайн тикетов целиком → `06_FUNCTIONS.md` |
| CI / автоматизация | `.github/workflows/*.yml` | 02, 03 (имена, триггеры) | частично | шаги и секреты (имена) → `19_INFRASTRUCTURE.md` |
| Скрипты проверок/генерации | `scripts/*.js` | 02, 03 (роли; читались `verify.js`, `generate-bank-csv.js`) | частично | инварианты `check-banks.js` → `20_VERIFICATION.md` |
| Агентные лейны | `.claude/agents/*.md`, `CLAUDE.md` осн. репо | 03, 28 (правила из `CLAUDE.md`) | частично | сами файлы агентов не читались → `21_AGENT_PROCESS.md` |
| Маркетинг | `docs/marketing/*` | 03 (перечень файлов) | не документировано | → `22_MARKETING_STATE.md` |
| SMM | `docs/smm/*` | 03 (перечень), 28 (механика очередей) | не документировано | → `23_SMM_STATE.md` |
| Email-шаблоны | `docs/email/`, `supabase/email-templates/` | 03 (перечень) | не документировано | — |
| Медиа-ассеты | `img/*`, иконки, `og-image.png`, флаеры `docs/marketing/flyer/` | 03 (перечень) | документировано (как состав) | — |
| Git-история | shallow-клон: 50 коммитов (2026-07-28…2026-08-04) | 01, 28 | частично | история глубже — UNKNOWN до полного клона → `26_CHANGELOG.md` |
| Ветка `content-banks-src` | банки, генераторы | 01, 03, 28 (состав, объёмы, la-business-law) | частично | различия скриптов между ветками не сверялись |
| Живые системы (Supabase prod, Stripe-аккаунт, GSC, соцсети) | вне репозитория | 28 («Ограничения сверки») | не документировано | данные фиксируются только при явной выгрузке владельцем; иначе UNKNOWN |

## Source References

- Документы `01`–`03`, `28` этого репозитория (их разделы «Source References»);
- полный листинг файлов `lalianamen/llicena@main` (357 файлов, `find`,
  2026-08-05) — как перечень областей, подлежащих покрытию;
- `git ls-tree origin/content-banks-src js/questions/`.

## Verification Status

**Verified** — таблица производна от созданных документов и фактического
листинга основного репозитория; утверждения о том, какие файлы читались,
а какие нет, соответствуют разделам «Verification Status» самих документов.
