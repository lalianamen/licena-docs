# INDEX — реестр документов базы знаний LICENA

Последняя сверка: 2026-08-30 (точечная: маркетинговые агрегаты; полная — 2026-08-05)
Обновляется при каждом создании или изменении документа (`CLAUDE.md` §9).

## Уровни достоверности

- **Verified** — каждое утверждение документа проверено чтением названного
  источника в основном репозитории.
- **Partially Verified** — часть данных помечена `UNKNOWN` и/или взята из
  описаний (header-комментариев, внутренних документов), не перепроверенных
  по реализации; такие места названы в «Verification Status» документа.
- **Unknown** — документ не проходил проверку.

## Реестр

| Документ | Статус заполнения | Последняя проверка | Достоверность |
|---|---|---|---|
| `CLAUDE.md` | заполнен | 2026-08-05 | n/a — правила, заданные владельцем, а не факты проекта |
| `README.md` | заполнен | 2026-08-05 | Verified |
| `01_PROJECT_OVERVIEW.md` | заполнен | 2026-08-05 | Partially Verified |
| `02_ARCHITECTURE.md` | заполнен | 2026-08-05 | Partially Verified |
| `03_REPO_STRUCTURE.md` | заполнен | 2026-08-13 | Partially Verified |
| `04_DATABASE.md` | заполнен | 2026-08-30 | Partially Verified |
| `05_API.md` | заполнен | 2026-08-05 | Partially Verified |
| `06_FUNCTIONS.md` | заполнен | 2026-08-30 | Verified |
| `07_AUTH_ACCESS.md` | заполнен | 2026-08-05 | Partially Verified |
| `08_SECURITY.md` | заполнен | 2026-08-30 | Partially Verified |
| `09_PAYMENTS.md` | заполнен | 2026-08-05 | Partially Verified |
| `10_CONTENT_MODEL.md` | заполнен | 2026-08-05 | Partially Verified |
| `11_I18N.md` | заполнен | 2026-08-05 | Partially Verified |
| `12_SEO.md` | заполнен | 2026-08-27 | Partially Verified |
| `13_UX.md` | заполнен (охватывает UX и Features) | 2026-08-29 | Partially Verified |
| `14_ANALYTICS.md` | заполнен | 2026-08-30 | Verified (по коду; часть живых данных подтверждена выгрузками владельца, первый запуск cron — UNKNOWN) |
| `15_METRICS.md` | заполнен | 2026-08-30 | Partially Verified |
| `16_PERFORMANCE.md` | заполнен | 2026-08-05 | Partially Verified |
| `17_TECH_DEBT.md` | заполнен | 2026-08-05 | Partially Verified |
| `18_DEPENDENCIES.md` | заполнен | 2026-08-30 | Verified |
| `19_INFRASTRUCTURE.md` | заполнен | 2026-08-13 | Partially Verified |
| `20_VERIFICATION.md` | заполнен | 2026-08-30 | Verified |
| `21_AGENT_PROCESS.md` | заполнен | 2026-08-05 | Partially Verified |
| `22_MARKETING_STATE.md` | не создан | — | — |
| `23_SMM_STATE.md` | не создан | — | — |
| `24_RISKS.md` | не создан | — | — |
| `25_DECISIONS.md` | не создан | — | — |
| `26_CHANGELOG.md` | заполнен | 2026-09-03 | Partially Verified |
| `27_ROADMAP.md` | не создан | — | — |
| `28_AI_CONTEXT.md` | заполнен | 2026-09-03 | Partially Verified |
| `29_BUSINESS_FRAMEWORK.md` | не создан | — | — |
| `30_CTO_REPORT.md` | не создан (будет создан пустым; заполняет ChatGPT) | — | — |
| `RELEASE_SUMMARY.md` | не создан | — | — |
| `rfc/` | не создана | — | — |
| `decisions/ADR-001-UNIFIED-COURSE-REGISTRY.md` | создан (статус решения: Proposed, не реализовано; путь задан владельцем 2026-08-05) | 2026-08-05 | Partially Verified (факты §1–3 — Verified; §4–13 — предложение) |
| `preview/roadmap/` | создана (превью-файлы фичи License Roadmap по команде владельца) | 2026-08-28 | Verified (см. README папки) |
| `tasks/` | создана (протокол владелец → ChatGPT → Claude: `WORKFLOW.md`, карточки задач, `reviews/`, `reports/`) | 2026-08-30 | Verified (журнал рабочего процесса; факты по живой БД — из выгрузок Supabase Dashboard, предоставленных владельцем) |
| `audit/INDEX.md` | заполнен | 2026-08-05 | Verified (производный от самих документов) |
| `audit/COVERAGE.md` | заполнен | 2026-08-05 | Verified (производный от документов + листинга основного репо) |

Пустые шаблоны не создаются намеренно (`CLAUDE.md` §2): «не создан» означает,
что документ появится, когда будет заполнен реальными данными.

## Source References

- Сами документы этого репозитория (их разделы «Verification Status»);
- `CLAUDE.md` этого репозитория (канонический список файлов 01–30).

## Verification Status

**Verified** — таблица отражает фактическое состояние файлов репозитория
licena-docs на дату сверки.
