# 12 — SEO-механика

Последняя сверка: 2026-08-05 (полная) · 2026-08-10 (точечная: HVAC-калькулятор)
Источник: `lalianamen/llicena@main`. Владелец области — лейн marketing
(`js/seo.js`, `sitemap.xml`, `robots.txt`, `docs/marketing/**` — `CLAUDE.md`
осн. репо). Реальные поисковые ЧИСЛА — в `15_METRICS.md`.

## Инжектор `js/seo.js` (594 строки; шапка и ключевые блоки прочитаны)

- Владеет crawl-facing `<head>` per page + per lang: `<title>`, description,
  canonical, robots, Open Graph, Twitter card, hreflang, JSON-LD. Загружается
  внешним файлом (CSP запрещает inline); JSON-LD — «данные, не исполняемый
  скрипт», поэтому проходит CSP (шапка файла).
- Вся SEO-копия — внутри файла (`COPY`, `ORG_DESC`, `COURSE_DESC`,
  `LIST_NAME`), трёхъязычно, НЕ в общих i18n; формулировки каждого языка
  нативные (`js/seo.js:56–63`).
- `PER_LANG_ROUTING = true` (owner decision 2026-07-14): `/` = EN,
  `/?lang=es`, `/?lang=ru` — canonical per-language + полный hreflang-кластер
  en/es/ru + `x-default` + `og:locale` с alternates. В коде «HONESTY GATE»:
  флаг валиден только пока роутинг реально работает (`js/seo.js:40–51`).
- Индексируемость: только лендинг (и статические страницы); `app.html` /
  `course.html` — noindex, несут только чистый локализованный `<title>`
  (`js/seo.js:60–62,79–86`).
- Принцип DOM-зеркала: structured data строится только из ВИДИМОГО контента —
  Course-список из `[data-seo-catalog]`, FAQPage из `[data-seo-faq]`,
  Breadcrumb из видимой навигации; нет секции — нет разметки, «markup can
  never drift from the page» (шапка + `js/seo.js:304–470`).

## JSON-LD узлы (schema.org)

| Где | Узлы |
|---|---|
| Лендинг | Organization (`sameAs`: Telegram, Facebook, Instagram, TikTok — только реально живые профили, `js/seo.js:29–39`), WebSite, ItemList курсов (Course-узлы с `@id` страницы экзамена — одна сущность с per-exam страницей), FAQPage |
| `/practice/<slug>/` | Organization, BreadcrumbList, Course, **Quiz** (`hasPart` из видимых sample-вопросов, `educationalAlignment`), FAQPage (`js/seo.js:387–525`) |
| `/guides/<slug>/` | Organization, BreadcrumbList, **Article** (headline/description из видимых h1/intro), FAQPage (`applyGuide`, `js/seo.js:494–515`) |

## Статические страницы: language-in-PATH (owner decision 2026-07-20)

Каждый пилотный экзамен/гайд — 3 реальных статических файла:
`/practice/<slug>/` (EN, canonical), `/es/practice/<slug>/`,
`/ru/practice/<slug>/`. Crawl-facing `<head>` на них «запечён» статически
(лучшая индексация + рабочие per-language share-карты, что схема `?lang=` дать
не могла); `js/seo.js` на этих страницах головы НЕ трогает — только JSON-LD
(`js/seo.js:120–131`). Аллоу-листы: `EXAMS` — 20 слагов, `GUIDES` — 19
(`js/seo.js:133–180`).

## sitemap.xml — 126 URL (пересчёт 2026-08-05)

| Секция | URL |
|---|---|
| Лендинг `/` + `/?lang=es` + `/?lang=ru` | 3 |
| `about` EN/ES/RU | 3 |
| `practice` 20 × EN/ES/RU | 60 |
| `guides` 19 × EN/ES/RU | 57 |
| `privacy.html`, `terms.html` | 2 |
| `app.html`/`course.html` (noindex) | не включены |

Дополнение 2026-08-10 (влито в `main` 2026-08-10 (fast-forward `ed30701`→`32f7201`)): +3 URL `/tools/hvac-sizing-calculator/`
(EN + `/es/`, `/ru/`) с полным hreflang-кластером; `grep -c "<url>"` = 128. Страницы `/tools/` несут полностью статический `<head>` и НЕ подключают
`js/seo.js`: его `detectPage()` знает только index/app/course плюс аллоу-листы
practice/guides — на любом другом пути инжектор перезаписал бы head данными
лендинга (`js/seo.js:231–236`, main-ветка `apply()`).

## robots.txt

Полностью процитирован в `02_ARCHITECTURE.md`: всё публичное открыто;
`Allow: /js/samples/` (нужны краулеру для рендера practice-страниц);
`Disallow: /supabase/ /docs/ /js/questions/ /js/guides/` («crawl-level
deterrent only»); app/course НЕ disallow'ятся сознательно — иначе краулер не
увидит их noindex; ссылка на sitemap.

## Прочая механика

- Google Search Console: верификация `googlec4665304f2eceeb0.html`; property
  подтверждена 2026-07-14 (`docs/marketing/gsc-readout-2026-08.md`).
- OG-image: `og-image.png` 1200×630, ~43 КБ (факт: 43 570 байт), flat brand
  colors, карточка `summary_large_image` (`js/seo.js:21–27`).
- `?v=`-версии на всех `<script src>` (сейчас `seo.js?v=14`) — инвалидация
  кэша при network-first SW; бампает design по спеке marketing
  (`CLAUDE.md` осн. репо; `gsc-readout` — история бампа v13→v14).
- «Free sample»-фрейминг practice-страниц после платного флипа (git `main`
  #187, 2026-08-04): заголовки типа «muestra gratis» / «пробный тест
  бесплатно» (образцы `<title>` зеркал — es/ru c-10).

## Маркетинговые SEO-документы (в `docs/marketing/`)

- `seo-audit-2026-07-19.md` (169 строк), `seo-backlog.md` (52),
  `spec-per-exam-pages.md` (494; спека per-exam страниц),
  `growth-narrow-first-2026-07.md` (319), `plan-2026-H2.md` (894),
  `decisions-log.md` (82) — полные тексты в этой сверке не читались
  (→ `22_MARKETING_STATE.md`); прочитан полностью
  `gsc-readout-2026-08.md` (130 строк) — первый data-driven SEO-проход:
  выводы «brand-first titles» (отгружено в v14), «ES — приоритетная лента»,
  «EPA 608 не монетизируется — контент/ссылки не наращивать», «страницы
  pos 40+ — проблема авторитета, не сниппетов». Итоговое состояние: все 20
  трейдов × EN/ES/RU practice + 19 гайдов × 3 языка построены — «future
  growth is depth + authority, not new page scaffolding».

## UNKNOWN

- Живые данные GSC после 2026-08-02; позиции/индексация на дату сверки.
- Поисковые объёмы запросов — в самом `gsc-readout` помечены UNVERIFIED
  (keyword-инструменты недоступны).
- Сделал ли design правку статического `<title>` в `index.html` из спеки
  gsc-readout: статический title = «LICENA — CSLB & California Contractor
  License Practice Tests» (прочитан, совпадает с новым) — выполнено;
  но три «bare gratis»-тайтла из раздела REVIEW (es/c-10, es/c-39,
  ru/c-36) — состояние правки не проверялось.

## Source References

- `js/seo.js` (строки 1–180, 219–304, 387–594 + grep-карта функций),
  `sitemap.xml` (пересчёт по секциям), `robots.txt`,
  `googlec4665304f2eceeb0.html`, `og-image.png` (размер файла),
  `index.html` (статический title), `es|ru/practice/c-10-electrical/`
  (образцы статических голов), git `main` #186–#187
- `docs/marketing/gsc-readout-2026-08.md` — полностью; остальные
  marketing-доки — только `wc -l` и назначение
- `CLAUDE.md` осн. репо (границы лейна marketing)

## Verification Status

**Partially Verified.**

- Проверено: структура и правила seo.js (по прочитанным блокам), состав
  sitemap (пересчёт), robots, аллоу-листы, статические головы (образцы),
  выполнение brand-first-правки в `index.html`.
- Прочитано частично: middle-секции `js/seo.js` (строки 305–479 — по grep,
  не построчно); marketing-доки кроме gsc-readout.
- UNKNOWN-позиции перечислены выше.
