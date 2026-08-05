# 21 — Агентный процесс (AI-работа над проектом)

Последняя сверка: 2026-08-05
Источник: `.claude/agents/*.md` (5 файлов, прочитаны: content/design/
resources/smm — головные разделы; marketing — головной + state-блок),
`CLAUDE.md` обоих репозиториев, `docs/content/bank-playbook.md`,
`.github/workflows/*`, `docs/content-audit/queue.json`. Только фактический
процесс, без оценок.

## Модель: оркестратор + 5 лейнов

Работу над основным репо ведут до пяти субагентов Claude; правило
бесконфликтности: **каждый редактирует только свои файлы**, оркестратор
сериализует деплои — диффы не пересекаются, merge в `main` без конфликтов
(`CLAUDE.md` осн. репо, «Agent lanes»). Все пять определений имеют YAML-
frontmatter (name/description/tools/`model: inherit`) и общий принцип:
**агент никогда не деплоит** — результат утверждает владелец.

| Лейн | Файлы-владения | Мандат (из файла агента) | Инструменты |
|---|---|---|---|
| **design** (120 строк) | HTML, `css/*`, презентационное стекло (`resources-render.js`), i18n лендинга | «решает И реализует» UI/UX; mobile-first, ESL-аудитория, «calm, credible, not flashy»; исполняет head-спеки marketing | Read/Grep/Glob/Edit/Write/Bash |
| **content** (130) | `js/questions/*`, `js/guides/*`, `js/course-ref.js`, `honest-chances`, `docs/content/**`, `js/bank-updates.js` | контент-аудитор: каждое утверждение — с официальным первоисточником; недоказанное — `UNVERIFIED`, «never guess»; текущая редакция источника | + WebFetch/WebSearch |
| **resources** (62) | ровно один файл `js/resources.js` (+ его i18n-ключи) | куратор официальных ссылок; ежемесячная ре-валидация каждой; «NEVER invents a URL» | + WebFetch/WebSearch |
| **marketing** (333) | `js/seo.js`, `sitemap.xml`, `robots.txt`, `docs/marketing/**` | ключевая позиция «does the site sell»: решает позиционирование/воронку/поиск; чужие файлы не трогает — выдаёт авторитетные спеки design/content; «Current state & owner decisions»-блок ведёт оркестратор; честность структурированных данных; no black-hat | + WebFetch/WebSearch |
| **smm** (128) | только `docs/smm/**` | пятый лейн: трёхъязычные пост-паки из проверяемых материалов репо; «никогда не постит сам — публикует владелец»; подчиняется стратегии marketing (читает её state-блок первым) | + WebFetch/WebSearch |

Иерархия решений (из файлов агентов): владелец → marketing (стратегия
продаж/каналов) → исполняющие лейны; smm не противоречит marketing;
факты текут из репо в посты, не наоборот.

## Распределение моделей (owner decision 2026-08-04)

Для производства банков — фиксированное:
генерация — топ-модель сессии (на момент записи — Fable); переводы RU/ES —
Opus; верификация — топ-модель в основном цикле. Записано в
`bank-playbook.md` §10 именно потому, что решение однажды потерялось при
компакции контекста чата.

## Автоматизированные конвейеры

1. **Тикет → PR → письмо** (полные тексты workflow прочитаны):
   чат-виджет пишет тикет → триггер БД создаёт GitHub issue
   (label `support`) → владелец ставит label **`from-chat`** (greenlight;
   гейт из-за стоимости и отсутствия rate-limit на чате) →
   `claude-code-action@v1` работает по CLAUDE.md на ветке
   `claude/ticket-<slug>` (≤50 turns, acceptEdits) и открывает PR, «NEVER
   merges» → владелец мержит → `claude-ticket-resolved.yml` вызывает
   `ticket-status` → триггер шлёт пользователю письмо «resolved» EN/ES/RU.
2. **Контент-аудит по крону**: очередь `docs/content-audit/queue.json` —
   каждые 5 часов cron берёт первый pending, субагент `content` проверяет
   банк по официальным источникам, «auto-deploy its grounded edits to main»,
   статус → done; политика «deploy-everything (autonomous): agent never
   fabricates; UNVERIFIED флагуется, не правится; каждый деплой — с
   one-command revert». Cron живёт в сессии и истекает через 7 дней;
   ежегодная ре-верификация перезапускается вручную (поле `_about`).
3. **Автопостинг SMM**: владелец кладёт готовый `.txt` в
   `docs/smm/queue/` (Telegram) или `queue-fb/` (Facebook) и пушит в main —
   workflow постит только ДОБАВЛЕННЫЕ файлы (`--diff-filter=A`); повторная
   отправка — `workflow_dispatch` с именем файла.

## Поведенческая база

- `CLAUDE.md` осн. репо: Karpathy-гайдлайны (думать до кода; минимализм;
  хирургические правки; verify-before-done с конкретным чек-листом) +
  инварианты банков + правила авторинга.
- `CLAUDE.md` licena-docs (этого репо): двойная роль Claude — разработка
  LICENA и фактическая документация; запрет выдумывания, `UNKNOWN`,
  append-only журналы, стандарт аудита §9; бизнес-оценки — только ChatGPT
  (`30_CTO_REPORT.md`).

## Разделение AI-ролей проекта (по утверждённым инструкциям)

| Роль | Кто | Основание |
|---|---|---|
| Разработка + фактическая документация | Claude (сессии + 5 лейнов + CI-агент) | оба `CLAUDE.md` |
| Бизнес-аудит, оценки, `30_CTO_REPORT.md` | ChatGPT | `CLAUDE.md` licena-docs §6 |
| AI-саппорт пользователей | Edge Function `assistant` (`claude-sonnet-4-6`) | `06_FUNCTIONS.md` |
| Утверждение и публикация всего | владелец | все 5 файлов агентов («NEVER deploys»), greenlight-label, ручной постинг SMM |

## UNKNOWN

- Фактическая частота/история запусков лейнов и cron-аудита (журналов
  запусков в репо нет; след — только коммиты и `bank-updates.js`).
- Полные тексты `marketing.md` (строки 61–333) и хвосты остальных
  агент-файлов — не читались построчно.
- Оркестратор: где физически живёт state-блок marketing и процесс его
  обновления — в репо описан только сам блок.

## Source References

- `.claude/agents/content.md`, `design.md`, `resources.md`, `smm.md`,
  `marketing.md` (головные разделы + state-блок)
- `CLAUDE.md` осн. репо («Agent lanes»), `CLAUDE.md` licena-docs
- `docs/content/bank-playbook.md` §10, `docs/content-audit/queue.json`
- `.github/workflows/claude-support.yml`, `claude-ticket-resolved.yml`,
  `telegram-post.yml`, `facebook-post.yml` — полностью
- `supabase/functions/ticket-issue/index.ts` (greenlight-обоснование)

## Verification Status

**Partially Verified.**

- Проверено: мандаты/владения/инструменты всех 5 агентов (по их файлам),
  конвейеры (полные тексты workflow + queue.json + триггеры), модельное
  распределение, поведенческая база.
- Не читалось построчно: тела агент-файлов целиком (см. UNKNOWN) —
  утверждения о них ограничены прочитанными разделами.
