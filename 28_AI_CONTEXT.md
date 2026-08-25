# 28 — AI Context: сводный контекст для AI-сессий

Последняя сверка: 2026-08-05 (полная) · 2026-08-11 (точечная: редизайн лендинга) · 2026-08-24 (точечная: Невада) · 2026-08-25 (точечная: мерж NV, старт генерации nv-cms)
Назначение файла: быстрый ввод в курс дела для любой новой AI-сессии.
Всё ниже — факты из `lalianamen/llicena` на дату сверки; оценок нет.

## Состояние проекта одним абзацем

LICENA (licena.us) — трёхъязычный (EN/ES/RU) тренажёр лицензионных экзаменов
США: статический сайт на GitHub Pages (ветка `main`), бэкенд Supabase
(Auth + Postgres/RLS + Edge Functions), платежи Stripe. Бесплатная бета
закончилась 2026-07-31; сейчас — подписка $20/мес за курс с разовым 3-дневным
триалом. Запущена одна витринная категория — Construction (Калифорния +
Аризона, 14 платных курсов и 7 бесплатных); банки Transportation и Beauty
написаны, но скрыты фильтром `LAUNCH_CATEGORIES` (`js/app-cabinet.js:72`).
Последний запуск — Arizona SRE (2026-08-04, первый курс вне Калифорнии).
Дополнение 2026-08-25: Невада (третий штат кабинета) ВЛИТА в `main`
владельцем (HEAD `961123d`) и живёт на licena.us — NV в `STATES` +
`js/catalog/nv.js` с 4 федеральными бесплатными банками (те же course id,
что в CA/AZ); анализ выполнимости — `docs/content/nv-feasibility.md`
(линейка nv-cms → nv-c2 → nv-c21 → nv-c1d → nv-b по бюллетеням PSI от
25.03.2026). Генерация платных NV-банков ИДЁТ (старт 2026-08-25): `nv-cms` —
блюпринт и fact ledger в `docs/content/nv-cms-blueprint.md`/`nv-cms-ledger.md`
(ветка осн. репо); **банк ГОТОВ полностью** (Run 6, 2026-08-25): на
`content-banks-src` — 500 вопросов × EN/RU/ES (ключи A125 B125 C125 D125,
финальный коммит `ec23475`; 10% выборка ключей перепроверена по
первоисточникам — 50/50), CSV для импорта сгенерирован
(`supabase/sql/bank_questions.csv`, 32,000 строк, gitignored); wiring на
ветке осн. репо (`beda7fa`): EXAM_FORMATS `"nv-cms" {count:60, passPct:75}`
+ EXAM_MINUTES 120, карточка курса в `js/catalog/nv.js` (подгруппа
`nv-nscb`), COURSE_REF-панель — ВЛИТО в `main` владельцем 2026-08-25
(fast-forward до `034521e`, задеплоено). Остался один шаг до пользователей:
владелец импортирует CSV в Supabase `bank_questions` (до импорта карточка
видна, банк пуст). Следующий банк линейки — `nv-c2` (Run 1 запланирован). Примечание к цифрам
абзаца: счёт курсов «14 платных / 7 бесплатных» — на дату сверки 2026-08-05;
релизы аризонских trade-банков 2026-08-15…08-19 в этот документ ещё не
внесены (см. пробел покрытия в `26_CHANGELOG.md`). Practice-страницы четырёх AZ trade-банков (`435946f`: 12 страниц +
`js/samples/az-*.js` + 4 slug в `EXAMS` + sitemap 137 URL) ВЛИТЫ в `main`
тем же fast-forward `034521e` (2026-08-25) и живут на проде.

## Ключевые инварианты (нарушение = сломанный прод)

1. **Деплой = push в `main`**: GitHub Pages отдаёт ветку как есть. Никаких
   секретов и платных банков в `main`.
2. **Платные банки не в `main`**: исходники — только ветка `content-banks-src`;
   сайт читает их из Supabase `public.bank_questions` (RLS). Правка платного
   банка: checkout ветки → правка + чекер там → `node
   scripts/generate-bank-csv.js` → владелец реимпортирует CSV в Supabase.
3. **Формы банков**: платные Construction — всегда 5 блоков × 100 (500);
   бесплатные сертификационные right-sized: `epa-608` 4×50, `osha-construction`
   5×50, `asbestos` 5×20 (owner decision 2026-07-16); остальные вертикали —
   50 вопросов / 6 секций.
4. **Переводы банков**: RU/ES — оверлеи по индексу; `opts` никогда не
   переупорядочивать (`correct` живёт только в EN-файле — реордер молча ломает
   ключи, рендер-проверка этого не увидит). EN+RU+ES правятся одним коммитом.
5. **Хуки DOM**: никогда не переименовывать/удалять `id`/`class`/`data-*`,
   которые читает JS другого файла (лейны редактируют разные файлы).
6. **После любой правки `js/questions/*`** — запускать `node
   scripts/verify.js`. Рендер-проверка обязательна: скриншоты desktop + 360px,
   EN + RU, клик по одному потоку; вопрос в плеере искать по `id`, не по
   номеру на экране (порядок перемешивается).
7. **Журнал `js/bank-updates.js`**: каждое изменение банка — честная запись
   (дата, банк, что сделано, источники, год данных); максимум 6 записей,
   новая вытесняет старейшую.
8. **Контент — только открытые официальные первоисточники**, формулировки
   с нуля; из реальных экзаменов и коммерческих банков не копировать.
   Генерация банков — строго по SOP `docs/content/bank-playbook.md`.
9. **Пять агентных лейнов** (design / content / resources / marketing / smm) —
   каждый правит только свои файлы; marketing не трогает чужие файлы, а выдаёт
   спеки; smm никогда не постит сам (правила — `CLAUDE.md` основного репо и
   `.claude/agents/*.md`).

## Где что лежит (карта навигации)

- Каталог курсов: `js/catalog.js` + `js/catalog/ca.js` / `az.js`
  (штаты TX/FL/NY заведены, `active:false`).
- Список платных курсов держится синхронно в **трёх** местах: `PAID` в
  `supabase/functions/stripe-checkout/index.ts`, `PAID_SUBS` в
  `js/app-cabinet.js`, `PAID_COURSES` в `js/app-course.js` (+ список в
  `scripts/generate-bank-csv.js`).
- Цена: `PRICE_CENTS = 2000` ($20/мес, owner decision 2026-07-14) — в
  `stripe-checkout`. Доступ выдаёт только `stripe-webhook` → `user_courses`.
- Триал: `supabase/sql/trial-3day.sql` (owner decision 2026-08-01), RPC
  `start_trial`, таблица `course_trials` (никогда не чистится — щит от
  фарминга).
- Анти-шеринг: `supabase/devices_anti_sharing.sql` — `register_device()`,
  5 устройств, swap ≤ 1 раза в 30 дней; клиент `js/devices.js` fail-open;
  лимит продублирован константой `MAX_DEVICES` в `js/i18n-app.js` (копия и код
  обязаны совпадать).
- SEO-голова: только `js/seo.js` (лейн marketing) + `sitemap.xml` + `robots.txt`;
  при правках HTML-страниц бампать `?v=` по спеке marketing.
- AI-саппорт: `supabase/functions/assistant/index.ts`, модель
  `claude-sonnet-4-6`; список фактов о продукте зашит в промпт функции —
  при изменении продукта его тоже надо обновлять.
- Автопостинг соцсетей: положить `.txt` в `docs/smm/queue/` (Telegram) или
  `docs/smm/queue-fb/` (Facebook) и запушить в `main` — workflow постит сам.
- Контент-аудит банков: очередь `docs/content-audit/queue.json`
  (статусы pending/done, политика «UNVERIFIED флагуется, не правится»).
- Детальные разборы в базе знаний: функции — `06_FUNCTIONS.md`; модель
  доступа — `07_AUTH_ACCESS.md`; безопасность (в т.ч. статусы
  security-todo) — `08_SECURITY.md`; платежи — `09_PAYMENTS.md`;
  контент-модель и SOP банков — `10_CONTENT_MODEL.md`; локализация —
  `11_I18N.md`; SEO — `12_SEO.md`; аналитика — `14_ANALYTICS.md`; реальные
  числа (включая первый GSC-экспорт 2026-07-17→08-02: 156 показов,
  4 клика) — `15_METRICS.md`; производительность — `16_PERFORMANCE.md`;
  техдолг — `17_TECH_DEBT.md`; зависимости — `18_DEPENDENCIES.md`;
  инфраструктура — `19_INFRASTRUCTURE.md`; проверки — `20_VERIFICATION.md`;
  агентный процесс — `21_AGENT_PROCESS.md`.

## Незапущенное / в работе (на 2026-08-05)

- Банк `la-business-law`: 500 EN + RU/ES-оверлеи готовы на `content-banks-src`
  (последние коммиты ветки), в каталоге и Stripe-карте отсутствует — не
  запущен. Что за экзамен и дата запуска — UNKNOWN.
- Армянский язык (`hy`): staged на курсе C-20 (`testLangs:["hy"]` в
  `js/catalog/ca.js` + `c20-exam.hy.js` на ветке банков) — виден только
  тестерам (`profiles.is_tester`).
- Тестерские триалы продлены до 31 октября
  (`supabase/sql/extend-tester-trials-oct31.sql`).
- (дополнение 2026-08-10, итог `abd8804`) HVAC-калькулятор: страницы
  `/tools/hvac-sizing-calculator/` EN/ES/RU (`js/hvac-calc.js`,
  `css/tools.css`) — COURSE-GATED: открываются только из плеера курса
  (`c20-exam`) через сворачиваемую секцию «Инструменты» в левом сайдбаре
  (свёрнута по умолчанию); клик ставит штамп `lp:tool_gate`, без свежего
  штампа страница редиректит на `/`; `noindex`, в sitemap НЕ входят, ссылок
  с лендинга/гайдов нет. Все секции справочной панели плеера теперь свёрнуты
  по умолчанию. Реестр `COURSE_TOOLS` per-course — калькуляторы добавляются
  любому банку одной записью; с `4234bff`+`6530354` — 10 калькуляторов
  (`TOOL_DEFS`: hvac, hvacdesign (Manual S/D для C-20), electrical,
  plumbing, roofing, concrete, painting, solar, fire, pricing) ТОЛЬКО на
  платных банках (решение владельца 2026-08-11: retention-ценность
  подписки; epa-608 и nicet удалены из реестра), 27 страниц `/tools/**`
  ×3 языка, общая логика `js/trade-calcs.js`. ВАЖНО: страницы `/tools/` не подключают
  `js/seo.js` (он не знает этот путь и перезаписал бы head данными
  лендинга); новые публичные пути либо добавлять в его детекторы, либо
  нести статический head без него.
- (дополнение 2026-08-11, `1e61960` → порт макета `04a2d99`) Лендинг
  `index.html` — точный порт утверждённого владельцем макета-синтеза
  Lovable-образцов (первую «эволюционную» версию владелец отклонил: «сделай
  по макету»). Тема — `css/landing.css` (грузится ТОЛЬКО index.html;
  переопределяет токены после `styles.css`, банды full-bleed через
  margin-breakout + `body{overflow-x:clip}`, auth-модал остаётся светлым);
  интерактив — `js/landing-extras.js` (калькулятор «сколько теряешь без
  лицензии»: ползунки + чипы-пресеты профессий, пресеты — примеры, НЕ
  статистика (BLS за egress-блоком; выдуманные цифры запрещены), живой
  вопрос из `js/samples/c10-exam.js` с двухкнопочным языковым пиллом);
  `js/i18n.js` v45. Секции proof/story и stats-band УДАЛЕНЫ (нет в макете;
  `syncProofShots` защищён if-ами). Hero-фото `img/hero-contractor.jpg`.
  Прежние хуки (`#navLogin`, auth-модал, `data-seo-*`, `testiSection`,
  support) сохранены; `js/seo.js`, sitemap, practice/guides не менялись.
  Переводы вердиктов/CTA, шаблонов окупаемости и единиц ($/час, /мес) JS
  читает из скрытых/встроенных `data-t`-элементов (`#lq-right`,
  `#gc-pb-*`, `.un`) — не переименовывать эти id/классы.

## Расхождения, зафиксированные при сверке (факты, без оценок)

- `README.md` основного репо описывает добета-состояние: «payments (Stripe)
  … still to come», «Free during the beta», упоминает язык `vi` и не упоминает
  каталог `js/catalog/`, Arizona, платные банки. Фактическое состояние — см.
  `01_PROJECT_OVERVIEW.md` и `02_ARCHITECTURE.md`.
- Комментарий в `scripts/generate-bank-csv.js` говорит «nine PAID banks», в
  самом списке `PAID` там 12 позиций, а платных курсов в Stripe — 14.
- `CLAUDE.md` основного репо говорит о «12 paid banks» на `content-banks-src`
  (запись от 2026-07-29); к 2026-08-05 платных курсов уже 14 (+ c33, az-sre).

## Ограничения этой сверки (UNKNOWN)

- Git-история глубже 50 последних коммитов (shallow-клон): дата старта
  проекта, полное число коммитов — UNKNOWN.
- Живые данные Supabase (число пользователей, строки таблиц, состояние
  секретов) — UNKNOWN: в репо только код и SQL, доступа к проду у сессии нет.
- Состояние Stripe-аккаунта (реальные подписки, выручка) — UNKNOWN.
- Данные Google Search Console — первый экспорт (2026-07-17→08-02)
  зафиксирован в `15_METRICS.md` по `docs/marketing/gsc-readout-2026-08.md`;
  более свежие данные GSC — UNKNOWN.

## Правила работы над этой базой знаний

См. `CLAUDE.md` этого репозитория: только реальные данные, `UNKNOWN` для
отсутствующих, обновление документации в том же цикле, что и код; append-only
журналы; `30_CTO_REPORT.md` ведёт ChatGPT — не редактировать; каждый документ
несёт разделы «Source References» и «Verification Status» (стандарт аудита,
`CLAUDE.md` §9).

## Source References

Все пути — в репозитории `lalianamen/llicena`:

- Инварианты и процессы: `CLAUDE.md` основного репо (лейны, формы банков,
  правила переводов, verify-чек-лист, ветка `content-banks-src`),
  `.claude/agents/` (листинг)
- Синхронные списки платных курсов: `supabase/functions/stripe-checkout/index.ts`
  (карта `PAID` + комментарий о `PAID_SUBS`/`PAID_COURSES`),
  `scripts/generate-bank-csv.js` (список `PAID`, строки 10–14)
- Цена/триал/бета: `supabase/functions/stripe-checkout/index.ts`
  (`PRICE_CENTS`), `supabase/sql/trial-3day.sql`,
  `supabase/sql/restore-beta-policies-until-aug1.sql`,
  `supabase/sql/extend-tester-trials-oct31.sql`
- Анти-шеринг: `supabase/devices_anti_sharing.sql` (по ссылке из
  `docs/access-control.md`), `js/devices.js`, `js/i18n-app.js`
  (`MAX_DEVICES`, комментарий о синхронизации с сервером)
- Витрина/каталог: `js/app-cabinet.js:72` (`LAUNCH_CATEGORIES`),
  `js/catalog.js`, `js/catalog/ca.js` (`testLangs:["hy"]`), `js/catalog/az.js`
- Контент: `js/bank-updates.js`, `docs/content/bank-playbook.md` (существование
  и роль — по `CLAUDE.md`), `docs/content-audit/queue.json` (`_about`, `policy`)
- Саппорт: `supabase/functions/assistant/index.ts` (`MODEL`, продуктовые факты
  в промпте)
- Автопостинг: `.github/workflows/facebook-post.yml`, `telegram-post.yml`
  (триггеры по путям `docs/smm/queue*/`)
- Расхождения: `README.md` основного репо, `scripts/generate-bank-csv.js:1`
  («nine»), `CLAUDE.md` основного репо («12 paid banks»)
- Незапущенное: `git log origin/content-banks-src` (la-business-law),
  `git ls-tree origin/content-banks-src js/questions/` (`c20-exam.hy.js`)
- История: git log `main` (50 коммитов shallow-клона)

## Verification Status

**Partially Verified.**

- Проверено чтением названных файлов: все инварианты, списки, константы, пути
  и расхождения.
- Обновление 2026-08-05 (этап 2): `bank-playbook.md`, `check-banks.js`, все 8
  Edge Functions, `docs/access-control.md`, `docs/security-todo.md` прочитаны
  полностью — соответствующие утверждения подтверждены исходниками
  (`06_FUNCTIONS.md`–`10_CONTENT_MODEL.md`).
- Позиции `UNKNOWN` перечислены в разделе «Ограничения этой сверки».
