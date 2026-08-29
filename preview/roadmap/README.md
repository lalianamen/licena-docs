# Превью фичи License Roadmap (Phase 1)

Последняя сверка: 2026-08-29
Файлы этой папки — превью-материалы фичи License Roadmap, снятые с кода
`lalianamen/llicena@main` (`2c71fc6` — версия с логикой персонализации;
первоначально собраны с `96f1465`). Владелец запросил хранение
превью-файлов в базе знаний (команда «закинь файлы в док
репозитарий», 2026-08-28); обновлены по команде «покажи превью»
(2026-08-29).

## Файлы

| Файл | Что это |
|---|---|
| `licena-roadmap-preview.html` | Самодостаточное превью: весь CSS/JS фичи (styles.css, roadmap.css, paths.js, roadmap-config.js, i18n-roadmap.js, app-roadmap.js) встроен в один HTML. Открывается локально в любом браузере без сервера; кнопки EN/ES/RU и «Демо-план» вверху. Без аналитики и Supabase — ответы живут в localStorage браузера. Этот же файл опубликован как Claude-артефакт. |
| `vercel-preview-index.html` | Исходник страницы публичного Vercel-превью: тянет стили/скрипты с licena.us (всегда показывает актуальную прод-версию фичи). |
| `artifact-demo-ru.png` | Скриншот демо-плана (RU, логика `2c71fc6`): рекомендация «оставшийся экзамен», строка позиции, 11 шагов. |
| `artifact-demo-en.png` | То же на EN. |
| `rm-landing-banner.png` | Баннер-вход «Get Your License Roadmap» на лендинге licena.us. |
| `rm-ru-intro-360.png` | Мобильный (360px) интро-экран RU. |

## Живые ссылки (на 2026-08-29)

- Прод: `https://licena.us/roadmap.html` (`?demo=1` — заполненный пример; `?lang=ru|es`)
- Vercel-превью (публичное, preview-деплой в проект masterdomhvac-site, production того проекта не затронут; актуальный деплой с `?v=` версии `2c71fc6`):
  `https://masterdomhvac-site-gvc5o6tam-armen-lalian-s-projects.vercel.app`
  (прежний деплой `…-bzoro4cjn-…` остаётся живым, но ссылается на старые `?v=`)
- Claude-артефакт (приватный по умолчанию, share — из меню страницы; версия `personalization-v2`):
  `https://claude.ai/code/artifact/b44537f8-c40e-4eac-98c7-df3b768ac681`

Описание самой фичи — `13_UX.md`; схема БД — `04_DATABASE.md`.

## Source References

- `lalianamen/llicena@main` `2c71fc6`: `roadmap.html`, `css/styles.css`,
  `css/roadmap.css`, `js/paths.js`, `js/roadmap/*` (содержимое встроено в
  `licena-roadmap-preview.html` дословно).
- Скриншоты `artifact-demo-*.png` — Playwright-рендер этих же файлов,
  2026-08-29; `rm-landing-banner.png` и `rm-ru-intro-360.png` — рендер
  с `96f1465` (2026-08-28; эти экраны в `2c71fc6` не менялись).

## Verification Status

**Verified** — файлы собраны скриптом из названных исходников и отрендерены
перед публикацией (0 ошибок консоли); ссылки проверены живыми 2026-08-29
(Vercel-превью — HTTP 200, отдаёт `app-roadmap.js?v=5`).
