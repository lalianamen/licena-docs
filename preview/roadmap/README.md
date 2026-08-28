# Превью фичи License Roadmap (Phase 1)

Последняя сверка: 2026-08-28
Файлы этой папки — превью-материалы фичи License Roadmap, снятые с кода
`lalianamen/llicena@main` (`96f1465`) в день выкладки. Владелец запросил
хранение превью-файлов в базе знаний (команда «закинь файлы в док
репозитарий», 2026-08-28).

## Файлы

| Файл | Что это |
|---|---|
| `licena-roadmap-preview.html` | Самодостаточное превью: весь CSS/JS фичи (styles.css, roadmap.css, paths.js, roadmap-config.js, i18n-roadmap.js, app-roadmap.js) встроен в один HTML. Открывается локально в любом браузере без сервера; кнопки EN/ES/RU и «Демо-план» вверху. Без аналитики и Supabase — ответы живут в localStorage браузера. Этот же файл опубликован как Claude-артефакт. |
| `vercel-preview-index.html` | Исходник страницы публичного Vercel-превью: тянет стили/скрипты с licena.us (всегда показывает актуальную прод-версию фичи). |
| `artifact-demo-ru.png` | Скриншот демо-плана (RU): 60%, «Ваш следующий шаг», «Вы здесь», 11 шагов. |
| `artifact-demo-en.png` | То же на EN. |
| `rm-landing-banner.png` | Баннер-вход «Get Your License Roadmap» на лендинге licena.us. |
| `rm-ru-intro-360.png` | Мобильный (360px) интро-экран RU. |

## Живые ссылки (на 2026-08-28)

- Прод: `https://licena.us/roadmap.html` (`?demo=1` — заполненный пример; `?lang=ru|es`)
- Vercel-превью (публичное, preview-деплой в проект masterdomhvac-site, production того проекта не затронут):
  `https://masterdomhvac-site-bzoro4cjn-armen-lalian-s-projects.vercel.app`
- Claude-артефакт (приватный по умолчанию, share — из меню страницы):
  `https://claude.ai/code/artifact/b44537f8-c40e-4eac-98c7-df3b768ac681`

Описание самой фичи — `13_UX.md`; схема БД — `04_DATABASE.md`.

## Source References

- `lalianamen/llicena@main` `96f1465`: `roadmap.html`, `css/styles.css`,
  `css/roadmap.css`, `js/paths.js`, `js/roadmap/*` (содержимое встроено в
  `licena-roadmap-preview.html` дословно).
- Скриншоты — Playwright-рендер этих же файлов, 2026-08-28.

## Verification Status

**Verified** — файлы собраны скриптом из названных исходников и отрендерены
перед публикацией (0 ошибок консоли); ссылки проверены живыми 2026-08-28.
