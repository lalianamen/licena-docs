# LICENA — протокол работы Owner → ChatGPT → Claude

Последняя сверка: 2026-08-29

## Назначение

Этот файл определяет обязательный цикл разработки LICENA через два репозитория:

- `lalianamen/licena-docs` — публичный слой постановки задач, отчётов, preview и review;
- `lalianamen/LLICENA` — приватный репозиторий реализации.

Приватный исходный код, secrets, PII и чувствительные implementation details не копируются в `licena-docs`.

## Роли

### Owner

- ставит продуктовую задачу ChatGPT;
- рассматривает выводы ChatGPT;
- принимает итоговое решение:
  - отправить Claude на коррекцию;
  - разрешить merge в private `main`;
  - остановить задачу.

### ChatGPT

- анализирует запрос Owner;
- при необходимости проверяет актуальные официальные источники;
- превращает запрос в конкретную проверяемую задачу для Claude;
- публикует задачу в `licena-docs/tasks/`;
- проверяет отчёт, preview, screenshots, tests/checks, migrations и риски;
- показывает Owner краткий вывод и рекомендацию;
- публикует `CHANGES_REQUESTED`, если результат требует доработки;
- никогда не разрешает merge без решения Owner.

### Claude

- самостоятельно отслеживает задачи в `licena-docs/tasks/`;
- берёт только задачи со статусом `READY_FOR_CLAUDE` или `CHANGES_REQUESTED`;
- переводит взятую задачу в `IN_PROGRESS`;
- изучает private `LLICENA` и его `CLAUDE.md`;
- реализует изменения только в отдельной private feature branch;
- не merge/deploy в private `main`;
- запускает предусмотренные проверки;
- для UI создаёт non-production preview и screenshots;
- публикует безопасный отчёт в `licena-docs`;
- переводит задачу в `READY_FOR_REVIEW`;
- ожидает решения ChatGPT и Owner.

## Канонический цикл

1. Owner ставит задачу ChatGPT.
2. ChatGPT анализирует задачу и публикует task-файл в `licena-docs/tasks/` со статусом `READY_FOR_CLAUDE`.
3. Claude обнаруживает задачу, меняет статус на `IN_PROGRESS` и выполняет её в private feature branch.
4. Claude публикует в `licena-docs`:
   - implementation summary;
   - private branch и commit SHA;
   - changed areas без private source;
   - tests/typecheck/build/check results;
   - migrations и их application status;
   - preview URL;
   - screenshots для UI;
   - official sources, если применимо;
   - unresolved issues и risks.
5. Claude ставит `READY_FOR_REVIEW`.
6. ChatGPT независимо анализирует результат и показывает Owner:
   - что сделано;
   - что подтверждено;
   - что не подтверждено;
   - найденные проблемы;
   - рекомендацию: approve, correction или stop.
7. Owner принимает решение.
8. Если нужны изменения, ChatGPT публикует точный correction request и ставит `CHANGES_REQUESTED`; Claude повторяет шаги 3–5.
9. Цикл повторяется до требуемого результата.
10. Только после явного решения Owner статус меняется на `APPROVED_FOR_MAIN`.
11. Merge/deploy выполняется отдельно и только после `APPROVED_FOR_MAIN`.
12. После подтверждённого merge/deploy и финальной проверки задача получает `DONE`.

## Статусы

| Статус | Значение |
|---|---|
| `READY_FOR_CLAUDE` | ChatGPT подготовил задачу; Claude может начать |
| `IN_PROGRESS` | Claude выполняет аудит/реализацию |
| `READY_FOR_REVIEW` | Claude завершил работу и опубликовал доказательства |
| `CHANGES_REQUESTED` | ChatGPT/Owner запросили конкретные исправления |
| `APPROVED_FOR_MAIN` | Owner явно разрешил merge |
| `DO_NOT_MERGE` | Merge запрещён |
| `DONE` | Разрешённый merge/deploy и финальная проверка завершены |

## Правила переходов

```text
READY_FOR_CLAUDE
  → IN_PROGRESS
  → READY_FOR_REVIEW
      → CHANGES_REQUESTED → IN_PROGRESS → READY_FOR_REVIEW
      → APPROVED_FOR_MAIN → DONE
      → DO_NOT_MERGE
```

- Claude не устанавливает `APPROVED_FOR_MAIN`.
- ChatGPT не устанавливает `APPROVED_FOR_MAIN` без явного решения Owner.
- `READY_FOR_REVIEW` не означает разрешение на merge.
- Наличие commit или preview не означает approval.
- Никакого автоматического merge или production deployment.

## Task-файл

Каждая задача должна содержать:

- цель и product context;
- scope и out-of-scope;
- safety constraints;
- discovery requirements;
- acceptance criteria;
- required checks;
- preview/screenshot requirements;
- migration rules;
- reporting requirements;
- текущий status;
- ссылки на предыдущий review/correction, если это повторный цикл.

## Отчёт Claude

Отчёт должен быть достаточным для независимой проверки ChatGPT, но безопасным для публичного репозитория.

Обязательные поля:

- task status;
- implementation summary;
- source branch и commit SHA;
- changed areas;
- test/check/build results;
- migration status;
- preview URL;
- screenshot links;
- unresolved issues;
- official sources;
- explicit statement: private `main` changed or unchanged;
- explicit statement: production deployed or not deployed.

Если сведения невозможно проверить, писать `UNKNOWN`.

## Preview

Для UI Claude обязан предоставить preview exact commit, а не старой версии:

- desktop;
- mobile;
- English;
- Russian или другой long-string locale;
- ключевые adaptive branches;
- zero-console-error result либо явный blocker.

Preview не является production deployment.

## Merge gate

Единственный допустимый gate:

1. Claude: `READY_FOR_REVIEW`;
2. ChatGPT: review completed;
3. Owner: явное согласие на merge;
4. task status: `APPROVED_FOR_MAIN`;
5. только после этого — merge;
6. post-merge verification;
7. `DONE`.

## Current task mapping

- Public task: `tasks/ADAPTIVE_LICENSING_ROADMAP.md`
- Private work item: `lalianamen/LLICENA#188`
- Current status: `IN_PROGRESS`
- Prepared implementation identified at private commit `037bdd908188e0f6559836323fee889197240d72`
- Required next action: Claude publishes a safe exact-commit handoff, preview and screenshots in `licena-docs`; ChatGPT then reviews it.
