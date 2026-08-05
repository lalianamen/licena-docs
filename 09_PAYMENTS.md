# 09 — Платежи (Stripe)

Последняя сверка: 2026-08-05
Источник: `lalianamen/llicena@main`. Схема таблиц — `04_DATABASE.md`;
устройство функций — `06_FUNCTIONS.md`; здесь — сквозная платёжная модель.

## Модель

- **Подписка $20/месяц за отдельный курс**, валюта USD
  (`PRICE_CENTS = 2000`, owner decision 2026-07-14 —
  `supabase/functions/stripe-checkout/index.ts:19–20`). Price создаётся
  на лету через `price_data` (recurring monthly), заранее созданных Stripe
  Price/Product в коде нет.
- **14 платных курсов** (SKU): `cslb-law`, `b-general-building`,
  `b2-residential-remodeling`, `c7-low-voltage`, `c8-concrete`, `c10-exam`,
  `c16-fire-sprinkler`, `c20-exam`, `c27-landscaping`, `c33-painting`,
  `c36-plumbing`, `c39-roofing`, `c46-solar`, `az-sre`.
- **Разовый 3-дневный триал без карты** перед подпиской (owner decision
  2026-08-01; RPC `start_trial`, «одна выдача навсегда» через
  `course_trials` — `supabase/sql/trial-3day.sql`).
- Всё остальное (7 бесплатных курсов Construction + скрытые вертикали) — вне
  платёжного контура.

## Хронология включения (по датированным файлам)

| Дата | Событие | Источник |
|---|---|---|
| 2026-07-14 | Решение о цене $20/мес; дата платного старта Aug 1, 00:00 PT | комментарии `stripe-checkout/index.ts:19`, `js/app-cabinet.js:628` |
| 2026-07-18 | Каркас подписок в БД (expires_at, auto_renew, stripe_subscription_id; cron экспирации) | `supabase/sql/subscriptions-schema.sql` |
| 2026-07-28 | Go/no-go: Stripe-аккаунт ACTIVE | шапка `supabase/sql/stripe-payments.sql` |
| до 2026-08-01 | Бета-строкам платных курсов проставлен `expires_at='2026-08-01T07:00:00Z'` | `subscriptions-schema.sql`, `stripe-payments.sql` A2/B2 |
| 2026-08-01 | Флип: RLS free-tier-only (самозапись платных запрещена), триал включён | `stripe-payments.sql` SECTION B; `trial-3day.sql`; `restore-beta-policies-until-aug1.sql` |

Клиентский гейт: `PAID_START_MS = Date.parse("2026-08-01T07:00:00Z")`;
`isPaidLive()` — время либо параметр `?paidtest=1` (сквозной тест
checkout→webhook→entitlement до флипа) — `js/app-cabinet.js:633–639`.

## Сквозной поток покупки

1. Кабинет: платный курс, триал уже использован → кнопка Subscribe →
   `startCheckout()` → `functions.invoke("stripe-checkout", {course_id})`
   (`js/app-cabinet.js:690–706`).
2. `stripe-checkout`: JWT-проверка по JWKS → отсечка живой подписки (409) →
   ленивое создание Stripe customer (`profiles.stripe_customer_id`) →
   Checkout Session (mode subscription; metadata `{user_id, course_id}` на
   сессии И подписке) → `{url}` (`stripe-checkout/index.ts:96–160`).
3. Браузер уходит на Stripe; возврат на
   `app.html?checkout=success|cancel&course=…`.
4. Stripe шлёт вебхук → `stripe-webhook` (подпись проверяется) →
   `checkout.session.completed`/`invoice.paid` → upsert `user_courses`:
   `status='active'`, `expires_at = конец оплаченного периода + 2 суток
   grace`, `auto_renew = !cancel_at_period_end`, `stripe_subscription_id`
   (`stripe-webhook/index.ts:37–63`). Это **единственный** писатель платных
   прав.
5. Кабинет после `checkout=success` опрашивает появление строки, записанной
   вебхуком («usually lands within a couple of seconds»), затем перерисовка
   (`js/app-cabinet.js:721–730`, `handleCheckoutReturn`).

## Жизненный цикл подписки

- **Продление**: `invoice.paid` двигает `expires_at` вперёд (та же upsert-
  ветка).
- **Отмена пользователем**: Stripe Customer Portal
  (`stripe-portal` → `{url}`; «cancel anytime» = `cancel_at_period_end`);
  `customer.subscription.updated` зеркалит `auto_renew`, даты доступа не
  трогаются — доступ до конца оплаченного периода.
- **Завершение**: `customer.subscription.deleted` → `status='inactive'`,
  `expires_at=now`. Параллельная страховка — ежедневный cron
  `expire-subscriptions` (00:30 PT) гасит просроченные строки; grace-буфер
  вебхука (2 суток) гарантирует, что медленное продление не даст ложной
  паузы (`stripe-webhook/index.ts:34–37`; `subscriptions-schema.sql` §3).
- **Пауза ≠ потеря данных**: прогресс сохраняется; карточка показывает
  Subscribe (`trial-3day.sql`, комментарий; `js/app-cabinet.js:817`).
- **Повторная покупка** истёкшей/отменённой подписки разрешена (проверка 409
  пропускает неактивные — `stripe-checkout/index.ts:117–121`).

## Триал

`start_trial(course_id)` (клиент: `js/app-cabinet.js:1168–1177`): список из
14 платных курсов зашит в функции; `'trial_used'` при повторной попытке —
строка `course_trials` переживает удаление курса; `'already_active'` при
живом доступе; успех — `user_courses.active` на 3 дня. UI-тексты:
«3 days free, no card — then $20/month…» (`js/i18n-app.js:56–61`).

## Синхронизируемые списки платных курсов (по коду — обязаны совпадать)

1. `PAID` — `supabase/functions/stripe-checkout/index.ts:26–41` (14);
2. `PAID_SUBS` — `js/app-cabinet.js:622–625` (14);
3. `PAID_COURSES` — `js/app-course.js:34–37` (14);
4. RLS-исключения — `stripe-payments.sql` SECTION B (14);
5. Список в `start_trial()` — `trial-3day.sql` (14);
6. `PAID` в `scripts/generate-bank-csv.js:10–14` (12 на `main`; 13 с
   `la-business-law` на ветке `content-banks-src` — этот список управляет
   только генерацией CSV, не продажей).
Комментарий в `stripe-checkout` прямо требует держать первые три в синхроне.

## Что невозможно проверить только по репозиторию (UNKNOWN)

- Живые данные: количество подписок, выручка, MRR, история платежей.
- Настройки Stripe Dashboard: включённые события вебхука (код ожидает 4),
  активация Customer Portal, налоговые настройки, брендинг квитанций,
  тестовый vs live режим ключей.
- Фактическое применение SECTION B и состояние RLS-политик в живой БД.
- Значения секретов (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`).

## Source References

- `supabase/functions/stripe-checkout/index.ts`, `stripe-webhook/index.ts`,
  `stripe-portal/index.ts` — полностью
- `supabase/sql/stripe-payments.sql`, `subscriptions-schema.sql`,
  `trial-3day.sql`, `restore-beta-policies-until-aug1.sql`,
  `extend-tester-trials-oct31.sql`
- `js/app-cabinet.js` (строки 622–639, 688–730, 796–817, 868–870, 982–994,
  1155–1177), `js/app-course.js:32–37`, `js/i18n-app.js:56–61, 247`
- `scripts/generate-bank-csv.js:10–14`; ветка `content-banks-src`
  (коммит d492ea0 — la-business-law в CSV-списке)

## Verification Status

**Partially Verified.**

- Проверено чтением кода: весь платёжный контур от кнопки до RLS — все три
  Stripe-функции, SQL, клиентские строки, синхронизируемые списки.
- UNKNOWN-позиции — раздел «Что невозможно проверить только по репозиторию»
  (это единственные непроверенные утверждения документа).
