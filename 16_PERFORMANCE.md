# 16 — Производительность

Последняя сверка: 2026-08-05
Правило документа: фиксируются только (а) фактические механизмы реализации,
подтверждённые кодом, и (б) реально измеренные значения. Замеров скорости
(Lighthouse, PageSpeed, Web Vitals) в репозитории НЕТ — grep по
`lighthouse|web vitals|pagespeed` во всех md/js/html/yml дал 0 совпадений;
все runtime-показатели — UNKNOWN.

## Механизмы (по коду)

| Механизм | Реализация |
|---|---|
| Отсутствие сборки | Vanilla HTML/CSS/JS без бандлера/минификации собственного кода; vendor (`supabase-js-2.110.0.js`) — минифицирован. Деплой = отдача файлов как есть с GitHub Pages |
| Service worker | network-first для same-origin GET с cache-fallback: онлайн-визитёры всегда получают свежие файлы, офлайн — последнюю загруженную версию; кэш `licena-v1`, precache только `/` и `/app.html`; кросс-доменное (Supabase, fonts) не перехватывается; non-GET не трогается (`sw.js`, шапка + код) |
| Инвалидация кэша | `?v=`-суффиксы на каждом `<script src>` (i18n-app v50, app-cabinet v56, seo v14 и т.д. — `app.html`); бамп CACHE-имени в sw.js чистит старый кэш при активации |
| Ленивая загрузка контента | Банк грузится только при открытии курса: бесплатный — `<script>` по id (`js/app-course.js:32–43`), платный — постранично из `bank_questions` (`.range()`, страницы по 1000 строк; банк 500 вопросов = 1000–1500 строк en+оверлей — `js/app-course.js:142–170`, комментарий) |
| Лёгкие SEO-страницы | `/practice/*` не грузят supabase-js вообще — бикон `js/pageview.js` через сырой fetch с `keepalive` («SEO landing pages that must stay light», шапка файла) |
| Шрифты | Google Fonts с `<link rel="preconnect">` к fonts.googleapis.com и fonts.gstatic.com (`index.html:32–33`); CSP разрешает только эти хосты |
| Изображения | og-image 43 570 Б (в коде seo.js помечен «~43 KB»); иконки: icon-512.png — 1 927 Б; правило «любой inline `<svg>` обязан иметь явный размер» — чек-лист `CLAUDE.md` осн. репо (после инцидента: «An unsized icon once shipped oversized to prod») |
| Ограничение объёма писем-отчётов | daily-stats читает сырые строки окна с оговоркой в коде «fine at our scale; revisit if views exceed ~50k» (`daily-stats/index.ts`) |
| Кэширование заголовков/CDN | Специальных cache-заголовков в репо нет — отдача стандартная для GitHub Pages; конфигурации CDN в репозитории нет |

## Измеренные размеры статики (репо; `du`/`ls`, 2026-08-05)

Полная таблица — `15_METRICS.md`. Ключевое для веса страниц:

- `index.html` 40 807 Б + CSS (styles.css из 148 КБ суммарных четырёх
  файлов) + JS лендинга (seo, pwa, vendor 204 КБ, supabase-client, stats,
  devices, i18n, app, reviews, support);
- `app.html` 22 815 Б + 19 скриптов (включая каталоги, i18n-app,
  honest-chances 1136 строк, course-ref не грузится в кабинете);
- practice-страница: HTML 24 КБ + сэмпл 28 КБ + seo/pwa/pageview (без
  vendor);
- статические банки в `main` — 4,9 МБ каталог, крупнейший файл 428 КБ
  (`backflow.ru.js`) — грузятся ТОЛЬКО в плеере соответствующего курса.

Фактический сетевой вес страниц (с учётом сжатия GitHub Pages), время
загрузки, LCP/CLS/INP — не измерялись: UNKNOWN.

## Косвенные наблюдения из внешних данных (не замер производительности)

Единственная связанная строка в измеренных данных проекта: GSC-отчёт
2026-08-05 — «Mobile already ranks much better than desktop (pos 11.3 vs
31.9) — nothing to fix, just confirms the pages render well on phones»
(`docs/marketing/gsc-readout-2026-08.md`). Это позиция в поиске, не метрика
скорости.

## UNKNOWN

- Lighthouse/PageSpeed/Web Vitals — не проводились (артефактов нет).
- Реальное время загрузки, TTFB GitHub Pages, эффективность SW-кэша.
- Поведение под нагрузкой: лимиты Supabase (rate limits, cold start Edge
  Functions) — в репо не зафиксированы.
- Размер и время ответа выборки платного банка из `bank_questions` в проде.

## Source References

- `sw.js`, `js/pwa.js`, `manifest.json`, `app.html`/`index.html`
  (`?v=`-версии, preconnect), `js/app-course.js:32–43,142–170`,
  `js/pageview.js`, `supabase/functions/daily-stats/index.ts`
  (масштабная оговорка), `js/seo.js:21–27` (og-image)
- Замеры: `du`/`ls` (2026-08-05); grep по perf-инструментам (0 совпадений)
- `CLAUDE.md` осн. репо (правило размеров SVG, инцидент с иконкой)
- `docs/marketing/gsc-readout-2026-08.md` (mobile/desktop позиции)

## Verification Status

**Partially Verified.**

- Механизмы и размеры файлов — проверены кодом и замерами (Verified-уровень).
- Все runtime-показатели — UNKNOWN (замеров в проекте не существует);
  документ это явно разделяет.
