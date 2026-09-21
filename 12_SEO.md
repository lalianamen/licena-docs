# 12 — SEO-механика

Последняя сверка: 2026-08-05 (полная) · 2026-08-10 (точечная: HVAC-калькулятор) · 2026-08-13 (точечная: Bing WMT + IndexNow) · 2026-08-27 (точечная: NV practice-страницы) · 2026-09-17 (точечная: +6 practice-страниц, allowlist `EXAMS` в `js/seo.js`) · 2026-09-20 (точечная: лендинг — пикер, каталог по штатам) · 2026-09-21 (точечная: шапка лендинга — четыре штата)

Дополнение 2026-08-27 (`a25d54e` осн. репо): добавлены 9 practice-страниц
трёх невадских платных банков — слаги `nv-cms-exam`, `nv-b-general-building`,
`nv-b2-residential` (EN + `/es` + `/ru`, статический `<head>` по образцу AZ);
`js/seo.js` EXAMS +3 слага (`?v=18` только на новых страницах — прецедент AZ,
на старых остаётся v=16/v=17); `sitemap.xml` 137 → 146 `<url>` (9 записей с
hreflang-набором, lastmod 2026-08-27); сэмплы `js/samples/nv-{cms,b,b2}.js` —
8 оригинальных трёхъязычных вопросов на банк (НЕ из платных банков; ключи
сверены дословно с NRS 624/108/338/455/608, модельными IRC/IBC 2018 в двух
адопциях и 29 CFR 1926), факт-плитки из PSI Nevada CIB (upd. 3/25/2026).

Дополнение 2026-08-12 (`e8211d3` осн. репо): датный переключатель платной
копии 1 августа в `js/seo.js` УДАЛЁН — платные описания (index ×3 языка,
`COURSE_DESC` без «Free during beta on LICENA») запечены в базовые строки;
`seo.js?v=15` на всех страницах (index/app/course + 117 practice/guides).
Мета-описания и JSON-LD страниц `/about/` ×3 больше не содержат
«Free until July 31, 2026» — заменены на текущий оффер.
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

Дополнение 2026-08-10 (итог, git `main` `abd8804`): страницы
`/tools/hvac-sizing-calculator/` переведены в course-gated режим — `noindex,
follow`, из `sitemap.xml` УДАЛЕНЫ (снова 125 `<url>`), ссылки с лендинга и из
гайдов сняты; страница открывается только из плеера курса (штамп
`lp:tool_gate`, см. `13_UX.md`). Головы страниц остаются статическими,
`js/seo.js` там по-прежнему не подключается — его `detectPage()` знает только
index/app/course плюс аллоу-листы practice/guides, на любом другом пути
инжектор перезаписал бы head данными лендинга (`js/seo.js:231–236`).

## robots.txt

Полностью процитирован в `02_ARCHITECTURE.md`: всё публичное открыто;
`Allow: /js/samples/` (нужны краулеру для рендера practice-страниц);
`Disallow: /supabase/ /docs/ /js/questions/ /js/guides/` («crawl-level
deterrent only»); app/course НЕ disallow'ятся сознательно — иначе краулер не
увидит их noindex; ссылка на sitemap.

## Прочая механика

- Google Search Console: верификация `googlec4665304f2eceeb0.html`; property
  подтверждена 2026-07-14 (`docs/marketing/gsc-readout-2026-08.md`).
- Bing Webmaster Tools: сайт добавлен вручную 2026-08-13 (импорт из GSC не
  сработал), верификация метатегом `msvalidate.01` в `index.html`
  (`c064543`); `sitemap.xml` подан, статус Processing (скриншоты владельца).
- IndexNow: ключ-файл `54435241f92a34641164ae1e98d2923c.txt` в корне +
  workflow `.github/workflows/indexnow.yml` (`05f0ab5`) — POST изменённых
  `*.html`-страниц на `api.indexnow.org` при каждом push в `main`;
  `app.html`/`course.html` (noindex) пропускаются.
- Пилот «золотой шаблон C-10» (`bf78c5a`+`8074a3e`, влит в `main`
  2026-08-13): EN `<title>`/OG/Twitter C-10 начинаются с «Free…»; ES/RU
  головы C-10 сознательно не тронуты (page-1 предупреждение gsc-readout);
  JSON-LD (Course+FAQPage+Breadcrumb) после перестановки блоков цел —
  проверено рендером; auth-CTA страницы несут `?src=c10smp`.
- Раскатка шаблона на все 20 трейдов (`21a6693`, 2026-08-13): EN-тайтлы
  всех practice-страниц ведут с «Free» (без дубля там, где Free уже был:
  asbestos, b-general, c-36, c-46, cslb-law); ES/RU головы не тронуты;
  `sitemap.xml` lastmod 60 practice-URL → 2026-08-13; auth-CTA несут
  per-trade `?src={trade}smp` (lawsmp, c20smp, epasmp, …).
- `llms.txt` в корне (`d181464`, конвенция llmstxt.org, экспериментально —
  чтение файла крупными AI-движками не подтверждено): фактическая сводка
  продукта (цены, бесплатные курсы, оригинальность вопросов) + 20
  practice- и 19 guide-ссылок (EN; зеркала /es/ /ru/ описаны прозой),
  все ссылки сверены со `sitemap.xml` скриптом при создании.
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
- `.github/workflows/indexnow.yml`, `54435241f92a34641164ae1e98d2923c.txt`,
  `index.html` (метатег `msvalidate.01`), `llms.txt` — дополнение 2026-08-13

## Verification Status

**Partially Verified.**

- Проверено: структура и правила seo.js (по прочитанным блокам), состав
  sitemap (пересчёт), robots, аллоу-листы, статические головы (образцы),
  выполнение brand-first-правки в `index.html`.
- Прочитано частично: middle-секции `js/seo.js` (строки 305–479 — по grep,
  не построчно); marketing-доки кроме gsc-readout.
- UNKNOWN-позиции перечислены выше.

## Аддендум 2026-09-12 — +9 practice-страниц и 4-й штат

Источник: ветка осн. репо `claude/keen-hamilton-nx0q8e`; `sitemap.xml` и
страницы прочитаны при внесении правок.

- **URL в `sitemap.xml` — 156** (было 147): +3 `c-54-tile`, +3
  `nv-c2-electrical`, +3 `nv-c21-refrigeration`, у каждого `lastmod 2026-09-12`,
  `changefreq monthly`, `priority 0.8` и четыре `xhtml:link` (en/es/ru/x-default).
  XML парсится (`xml.etree`).
- **Practice-страниц — 30 экзаменов × 3 языка** (было 27). Новые страницы
  собраны из существующих шаблонов (C-33 для C-54, Nevada B для обоих NV),
  поэтому общая практическая UX, оффер и дисклеймеры идентичны остальным;
  `scripts/check-offer.js` подтверждает канон оффера на 72 платных страницах
  (было 63).
- **Заголовки и описания** новых страниц — в трёх языках, с брендом в конце
  `<title>` (шаблон existing-страниц), canonical + три hreflang + x-default,
  og/twitter синхронизированы со статикой страницы.
- **Роадмап-страницы** в `sitemap.xml` по-прежнему отсутствуют (закрыты от
  индексации намеренно); новая `roadmap-la.html` туда тоже не добавлялась.

### Verification Status (аддендум 2026-09-12)

**Verified** — числа получены прямым подсчётом в `sitemap.xml` и выводом
`check-offer.js`; рендер новых страниц проверен Playwright (1280/390/360,
EN/ES/RU, ноль ошибок консоли).

## Аддендум 2026-09-17 — +6 practice-страниц (la-building, A) и allowlist `EXAMS`

Источник: ветка осн. репо `claude/question-bank-generation-analysis-y47sk7`
(`732ab12`, `fe62cf7`), ЖДЁТ мержа владельцем; `js/seo.js`, `sitemap.xml` и
страницы прочитаны при внесении правок.

- **URL в `sitemap.xml` — 167** (было 161): +3 `la-building-construction`, +3
  `a-general-engineering`, у каждого `lastmod 2026-09-17`, `changefreq monthly`,
  `priority 0.8` и четыре `xhtml:link` (en/es/ru/x-default). XML парсится
  (`xml.dom.minidom`).
- **Practice-страниц — 33 экзаменов × 3 языка** (было 31: 30 на 2026-09-12 плюс `la-business-and-law`, `lastmod 2026-09-13`). Страницы Луизианы
  собраны из шаблона `la-business-and-law`, страницы A — из `c-8-concrete`
  (дисклеймер CSLB/PSI сохранён); `scripts/check-offer.js` подтверждает канон
  оффера на 81 платной странице (было 75).
- **Allowlist `EXAMS` в `js/seo.js` — исправлена ошибка прода.** Механика:
  `detectExam()` признаёт practice-страницу только по slug'у из `EXAMS`; иначе
  `apply()` идёт по ветке главной страницы и перезаписывает `document.title`,
  `<link rel="canonical">` и hreflang на `https://licena.us/` (статический
  `<head>` при этом остаётся в HTML, но краулер, исполняющий JS, видит
  подменённые значения). До 2026-09-17 в списке не было четырёх опубликованных
  slug'ов — `la-business-and-law`, `nv-c2-electrical`, `nv-c21-refrigeration`,
  `c-54-tile` (12 страниц) — плюс два новых. Добавлены все шесть; `seo.js?v=19`
  на 18 страницах. Влияние на индексацию этих 12 страниц за прошедший период —
  UNKNOWN (по GSC не проверялось). Автоматической проверки «каталог
  `/practice/*` ⊆ `EXAMS`» в репозитории НЕТ.
- **JSON-LD** новых страниц — стандартный набор practice-страницы
  (Organization, BreadcrumbList, Course, Quiz из 8 видимых вопросов, FAQPage из
  7 вопросов), заголовок и описание в трёх языках с брендом в конце `<title>`,
  canonical + три hreflang + x-default, og/twitter синхронизированы со статикой.

### Verification Status (аддендум 2026-09-17)

**Verified** — числа получены прямым подсчётом в `sitemap.xml`, `ls practice/`
и выводом `check-offer.js`/`verify.js`; поведение allowlist воспроизведено в
Playwright до и после правки (title страницы `la-building-construction` до
правки = заголовок главной); рендер 18 комбинаций (2 slug'а × EN/ES/RU ×
1280/390/360) — ноль ошибок консоли, клик-проход ответ → Далее → Назад.
**UNKNOWN** — влияние подмены canonical на индексацию 12 ранее опубликованных
страниц.

## Аддендум 2026-09-20 — лендинг: первый экран, пикер, каталог по штатам (ветка, ждёт мержа)

Источник: ветка осн. репо `claude/question-bank-generation-analysis-y47sk7` (`4e2b97d`).

- **Статический EN-текст `index.html` синхронизирован с `js/i18n.js`** (27 элементов `data-t`
  расходились: HTML показывал старые формулировки краулерам без JS — например «The Law & Business
  bank … included with every account», которого в i18n уже не было).
- **`#exams` (`data-seo-catalog`)**: карточки сгруппированы по штатам, добавлены 14 отсутствовавших
  (A, C-54, AZ ×4, NV ×5, LA ×2); `data-seo-course`/`data-seo-name` сохранены — JSON-LD ItemList
  теперь перечисляет все 33 экзамена. Ссылки карточек и пикера при `?lang=es|ru` переписываются на
  языковые страницы; `LICENA_SEO.apply()` вызывается после переключения, так что список отражает
  язык.
- Новый блок `#pick` (без SEO-хуков), секции `#pain`, `#method`, `#cost` и `.offer-table` удалены;
  FAQ (`data-seo-faq`) +2 вопроса → FAQPage JSON-LD растёт до 9 записей.
- Title/description/OG лендинга не менялись (в них по-прежнему «CSLB & California…» — при
  следующем SEO-проходе привести к четырём штатам; вне этой задачи).

### Verification Status (аддендум 2026-09-20)
**Verified** — прочитан diff `index.html`, Playwright: `#exams a[href*="/practice/nv-cms-exam/"]`
= `/ru/practice/…` при `?lang=ru`; 0 ошибок консоли. **UNKNOWN** — влияние на индексацию.

## Аддендум 2026-09-21 — шапка лендинга на четыре штата (ветка, ждёт мержа)

Источник: ветка осн. репо `8ea3685`.

- **Расхождение, которое закрыто.** После конверсионного прохода 2026-09-20 страница
  продавала четыре штата, а `<title>` и `description` (и статические, и инжектируемые
  `js/seo.js`) остались калифорнийскими: «LICENA — CSLB & California Contractor License
  Practice Tests». Это был текст сниппета в выдаче.
- **Новые значения** (EN/ES/RU, в `COPY.index` и в статической шапке `index.html`, они
  синхронны): title «LICENA — Contractor License Practice Tests: CSLB, Arizona, Nevada,
  Louisiana» и переводы; description называет четыре штата, три языка, бесплатный пример
  из 8 вопросов без регистрации и цену.
- **Заодно обновлены**: `ORG_DESC` (описание Organization уходит в JSON-LD КАЖДОЙ
  страницы) и `LIST_NAME` (имя ItemList каталога на лендинге).
- **Cache-bust**: `seo.js?v=20` на всех 163 страницах — прежде версии расходились
  (16/17/18/19), теперь одна.
- Заголовки practice-страниц не менялись: они пер-экзаменные и уже точны.

### Verification Status (аддендум 2026-09-21)
**Verified** — Playwright: лендинг `/`, `/?lang=es`, `/?lang=ru` и `/practice/c-20-hvac/`;
title и description соответствуют новым строкам, JSON-LD содержит Organization с новым
описанием и ItemList из 33 экзаменов с новым именем, 0 ошибок консоли.
