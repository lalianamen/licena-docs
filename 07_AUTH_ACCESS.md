# 07 — Аутентификация и модель доступа

Последняя сверка: 2026-08-05
Источник: `lalianamen/llicena@main`. Схемы таблиц и политики RLS подробно —
в `04_DATABASE.md`; здесь — сквозная модель: кто и как получает доступ.

## Аутентификация (Supabase Auth)

- Email + пароль; регистрация через `supa.auth.signUp` с metadata
  `{name, lang}` (`js/app.js:202–205`).
- Клиентская валидация регистрации (`js/app.js:176–222`): имя обязательно;
  email по регэкспу `EMAIL_RE`; пароль — `passOk()`: минимум одна заглавная
  буква и один символ (`js/app.js:32–34`); повтор пароля; обязательный
  чекбокс согласия. Серверная политика длины пароля Supabase Auth — UNKNOWN
  (настройка дашборда; `docs/security-todo.md` п.5 от 2026-06-24 упоминал
  дефолт min 6).
- Подтверждение почты обязательно: «уже зарегистрирован» показывает ТОТ ЖЕ
  экран «проверьте почту», что и новая регистрация — защита от перечисления
  аккаунтов (`js/app.js:206–210`, комментарий); `mailer_autoconfirm: false`
  отмечен как проверенный на живом проекте в `docs/security-todo.md`
  («What's already solid», аудит 2026-06-24).
- Данные профиля (имя, язык) до первого входа хранятся в localStorage
  (`lp:pending_name`, `lp:pending_lang`) — запись в `profiles` требует
  сессии из-за RLS (`js/app.js:210–212`).
- Сброс пароля: `doForgot` (`js/app.js:229+`); письма — шаблоны
  `supabase/email-templates/confirm-signup.html`, `reset-password.html`.
- Auth flow — **implicit**: токен в URL письма, чтобы ссылки подтверждения/
  сброса работали в другом браузере, чем регистрация (PKCE требовал бы
  code_verifier исходного браузера) — `js/supabase-client.js`, комментарий.
- Single-session-per-user: комментарий в
  `supabase/functions/stripe-checkout/index.ts:60–65` описывает режим, при
  котором новый логин убивает сессию старой вкладки (`/auth/v1/user` отвечает
  403 `session_not_found`), поэтому Stripe-функции проверяют JWT по подписи
  (JWKS), а не по живой сессии. Сама настройка single-session — в дашборде
  Supabase, из репозитория не видна (UNKNOWN: включена ли).

## Доступ к курсам (entitlements)

Единица доступа — строка `user_courses(user_id, course_id, status,
expires_at, auto_renew, stripe_subscription_id)` (`04_DATABASE.md`).

Пути получения строки:

| Путь | Кто пишет | Условия |
|---|---|---|
| Бесплатный курс | сам клиент (upsert) | RLS-политика `courses: own insert (free tier only)` — только course_id вне списка 14 платных (`supabase/sql/stripe-payments.sql` SECTION B) |
| Автовыдача новым аккаунтам | клиент (`contractor-business`, `epa-608`) | комментарий `supabase/sql/backfill-business-guide.sql`; существующим — бэкфилл тем же файлом |
| 3-дневный триал платного курса | RPC `start_trial()` (security definer) | один раз на (user, course) навсегда; `expires_at = now()+3 days` (`supabase/sql/trial-3day.sql`) |
| Платная подписка | только Edge Function `stripe-webhook` (service role) | после `checkout.session.completed` / `invoice.paid`; `expires_at` = конец периода + 2 суток grace (`stripe-webhook/index.ts`) |
| Тестерский доступ | владелец вручную SQL-скриптом | `extend-tester-trials-oct31.sql` (до 2026-11-01), `tester-account.sql` |

Прекращение доступа: ежедневный cron `expire-subscriptions` (00:30 PT) флипает
в `inactive` строки с истёкшим `expires_at`
(`supabase/sql/subscriptions-schema.sql` §3); отмена подписки — вебхук
`customer.subscription.deleted`. Прогресс при паузе сохраняется (комментарий
`trial-3day.sql`).

Проверка доступа в рантайме:

- Плеер при открытии курса читает свою строку `user_courses`
  (`js/app-course.js:97,734`); платный неактивный курс → пейвол.
- Данные платных банков дополнительно защищены на уровне строк:
  `bank_questions` RLS отдаёт строки только владельцу АКТИВНОЙ строки курса;
  чужой/анонимный запрос возвращает пусто без ошибки
  (`supabase/sql/bank-questions-schema.sql`; `js/app-course.js:48–56`,
  комментарий). Т.е. гейтится не только UI, но и сами данные.
- Кабинет различает «триал доступен / триал использован / подписка» по
  `course_trials` (RLS own select) + `stripe_subscription_id`
  (`js/app-cabinet.js:660–668, 868–870, 982–994`).

## Анти-шеринг устройств

Реализация (действующая): серверная функция `register_device(p_token,
p_confirm)` — 5 устройств на аккаунт, при заполнении замена не чаще 1 раза в
30 дней с вытеснением самого неактивного; тестеры без лимита; политики
insert/delete на `devices` сняты — записи только через функцию
(`supabase/devices_anti_sharing.sql`, полный контракт — `04_DATABASE.md`).
Клиент `js/devices.js` — тонкий вызов, **fail-open**: сбой RPC не блокирует
вход. Устройство = случайный первопартийный токен `lp:device` (не
IP, не fingerprint). Лимит продублирован в UI-константе `MAX_DEVICES`
(`js/i18n-app.js:2–4`, комментарий «copy and code must agree»).

Историческая справка: первая часть `docs/access-control.md` описывает ДРУГУЮ,
нереализованную модель («activation code → device binding», таблицы
`subscriptions`/`access_codes`) и прямо помечена в самом документе как
«the future anti-sharing design»; фактически построенное описано в его
разделе «Course entitlements — what is actually built today» и с тех пор
дополнено `register_device` (2026-07-20) и Stripe-вебхуком (2026-08-01).

## Разделение ролей ключей

- Браузер: только publishable/anon key (`js/supabase-client.js`); все records
  видимы строго через RLS.
- Service role: только внутри Edge Functions и cron (`stripe-webhook`,
  `assistant` — запись тикетов, `ticket-status`, `daily-stats`) и в
  дашборд-операциях владельца; в репозитории и git-истории ключа нет
  (проверка отмечена в `docs/security-todo.md`, «What's already solid»).
- Внутренние вебхуки БД→функции авторизуются приватным `x-ticket-secret`;
  publishable key в заголовке — только для прохождения Edge-гейтвея
  (`supabase/ticket_*_trigger.sql`, комментарии).

## Source References

- `js/app.js` (строки 32–34, 176–222, 229+), `js/supabase-client.js`,
  `js/devices.js`, `js/i18n-app.js:2–4`, `js/app-cabinet.js` (строки
  622–639, 660–668, 868–870, 982–994, 1155–1173), `js/app-course.js`
  (строки 32–56, 97, 734)
- `docs/schema.sql`, `docs/access-control.md`, `docs/security-todo.md`
- `supabase/devices_anti_sharing.sql`, `supabase/sql/stripe-payments.sql`,
  `trial-3day.sql`, `subscriptions-schema.sql`, `bank-questions-schema.sql`,
  `tester-account.sql`, `extend-tester-trials-oct31.sql`,
  `backfill-business-guide.sql`
- `supabase/functions/stripe-webhook/index.ts`, `stripe-checkout/index.ts`
- `supabase/email-templates/confirm-signup.html`, `reset-password.html`

## Verification Status

**Partially Verified.**

- Проверено чтением кода: вся клиентская валидация, контракты RPC, RLS-модель
  доступа к банкам, пути выдачи/прекращения доступа, JWKS-проверка.
- Взято из внутренних документов без независимой проверки: пункты
  `docs/security-todo.md` «verified live» (датированы 2026-06-24).
- UNKNOWN: серверные настройки Supabase Auth (мин. длина пароля,
  single-session, срок жизни токенов), фактическое применение SECTION B к
  живой БД.
