# REPORT — Adaptive Roadmap: даты G1–G4 реализованы (ветка a8553ba)

Задача: `tasks/ADAPTIVE_LICENSING_ROADMAP.md` («Dates as first-class data») · Dispatch: `lalianamen/LLICENA#188`
Дата: 2026-08-29 · Автор: Claude (интерактивная сессия владельца)
Private branch: `claude/ticket-adaptive-licensing-dates` @ `a8553ba` (база — `main` @ `037bdd9`)

## Контекст передачи

Discovery-отчёт claude[bot] в #188 (05:41 UTC) назвал 4 gap'а и объявил имплементацию на этой ветке, но за 2 часа ветка не была запушена (ни коммитов, ни PR, ни комментариев). По правилу координации работа подхвачена интерактивной сессией; имя ветки сохранено для преемственности ревью.

## Явные заявления

- **Private `main` — UNCHANGED.** Вся работа на feature-ветке; merge/deploy — только после APPROVED_FOR_MAIN.
- **Production database — UNCHANGED**; `license-roadmaps-v3.sql` НЕ применялся.
- Прод продолжает обслуживать `037bdd9`.

## Что сделано (гэпы из discovery)

| Gap | Реализация |
|---|---|
| **G1** — нет даты принятия | Вопрос «When did CSLB accept it?» при статусе Accepted; колонка `application_accepted_at` (v3 SQL); зеркалится из answers jsonb |
| **G2** — невидимы 2 жёстких дедлайна CSLB | Выводятся (не хранятся): `corrAt + 90 дней` и `appAcceptedAt + 18 месяцев`; оба участвуют в NEXT IMPORTANT DATE; **активный дедлайн в прошлом показывается как OVERDUE** (красная ячейка «N days OVERDUE»), а не пропускается; на карточках review/exams — заметки с датой пользователя и официальным источником. Правила перепроверены live 2026-08-29: [Application_Returned.aspx] «resubmit … within 90 days … becomes void», [Examination_FAQ.aspx] «18 months from the day they are accepted»; источники с датами — в `roadmap-config.js` |
| **G3** — не собирается номер заявления | Необязательное поле «Application Fee #» (термин — как в официальном чекере статуса); показывается на карточке review рядом со ссылкой проверки статуса; колонка `application_number` |
| **G4** — нет времени/места событий | Необязательное свободное поле «Time & place» для Live Scan и каждого экзамена; колонки `fingerprint_where` / `law_exam_where` / `trade_exam_where` |

Изменённые области: `js/roadmap/{app-roadmap,i18n-roadmap,roadmap-config}.js`, `css/roadmap.css`, `?v=`-пины 4 HTML, новый `supabase/sql/license-roadmaps-v3.sql` (идемпотентный, rollback в комментарии). 9 файлов, +172/−18. Обратная совместимость: все новые поля optional; клиент падает обратно на прежнюю форму строки до применения v3 (и далее до v2/v1 — каскад сохранён).

## Verification

| Проверка | Результат |
|---|---|
| Прежний сьют персон A–J (полный путь анкеты, все ветвления) | **74/74 PASS** — регрессий нет |
| Новый сьют дат: K — коррекция 100 дней назад → «Correction deadline … 10 days OVERDUE» + красная ячейка + заметка 90 дней с источником + номер заявления на карточке; L — принято 17 мес назад, trade не сдан → «Exam window ends … 38 days remaining» + заметка с FAQ-источником; M — время/место Live Scan и Law-экзамена сохраняются, ближайшая будущая дата побеждает; RU 360 — «ПРОСРОЧЕНО на 5 дн.» без h-scroll | **20/20 PASS** |
| `node scripts/verify.js` | 0 ошибок, 138 JS OK |
| i18n-паритет | 334 ключа/язык, все ссылки кода разрешаются |
| Рендеры просмотрены | скриншоты ниже |

## Скриншоты (non-production, локальный рендер ветки)

- `tasks/reports/img/adaptive-v3-overdue-en.png` — дашборд с просроченным 90-дневным дедлайном (EN, desktop)
- `tasks/reports/img/adaptive-v3-overdue-ru-360.png` — «ПРОСРОЧЕНО» (RU, 360px)

## Миграция

`supabase/sql/license-roadmaps-v3.sql` — 5 колонок `ADD COLUMN IF NOT EXISTS`, применяется ПОСЛЕ v2, только после APPROVED_FOR_MAIN. Статус применения: **не применялся**. RLS не меняется.

## Статус

Для этой ветки: **READY_FOR_REVIEW**. Общая задача: обе части (аудит `037bdd9` + даты `a8553ba`) теперь в READY_FOR_REVIEW — решение за ChatGPT/владельцем.
