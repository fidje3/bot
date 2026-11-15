# WoWBOX — Telegram-бот + Mini App

Готовый репозиторий для GitHub и Railway: Python-бот на aiogram, Mini App на Next.js, PostgreSQL через Drizzle, Stars Invoice, ручные RUB/USDT-заявки, проверяемые кейсы, апгрейд, инвентарь и отменяемые выводы.

## Что реально работает

- **25 кейсов** от ежедневного до Genesis; доля подарков растёт с ценой от 10% до 88%, в кейсах расширенная NFT-витрина.
- **Точный NFT:** бесплатный одноразовый кейс с гарантированным NFT после проверки подписки @WoWB0X, 100 оплаченных Stars и 4 новых рефералов.
- **Игровой зал:** Dice, Rocket и Wheel с отдельными ставками, выплатами, анимациями и серверными Provably Fair раундами.
- **Админ:** Telegram ID `6658790377` (@WoWBOXadmin) имеет бесконечный игровой баланс на серверном уровне.
- 70+ коллекций Telegram Gifts, Telegram Premium, обычные подарки в LIVE и Telegram Stars вместо декоративной валюты.
- **Единые шансы на клиенте и сервере.** Вероятности видны до открытия, сумма равна 100%. Рулетка проигрывает уже полученный серверный результат.
- **Provably Fair:** SHA-256 hash активного server seed, HMAC-SHA256 от `clientSeed:nonce:action`, roll 0–999999, раскрытие использованного seed и проверка последнего раунда.
- **Ежедневный бесплатный кейс** по UTC с несколькими диапазонами баланса, включая редкие крупные начисления.
- **Upgrade:** до 10 предметов, цель дороже входа, автоматический шанс, лаймовый сектор колеса равен шансу, компенсация 4% и временный бонус +25% при неудаче.
- **Инвентарь:** атрибуты модели/фона/символа, серийный номер, продажа за 70% внутренней оценки, история статусов.
- **Вывод:** NFT или заявка на Stars (85% оценки), блокировка предметов, отмена и разблокировка, команды оператора.
- **Пополнение:** Telegram Stars через официальный `createInvoiceLink` / `pre_checkout_query` / `successful_payment`; RUB и USDT TRC-20 через проверяемые оператором заявки.
- **LIVE:** последние реальные открытия из БД + явно подписанная случайная системная витрина, не выдающая себя за других пользователей.
- **Безопасность:** проверка подписи Telegram initData, серверные секреты, транзакции и row locks, уникальный charge ID, защита внутренних API.
- **Документы:** `/legal/terms`, `/legal/privacy`, команды бота `/terms`, `/privacy`.
- **TelegramGifts:** `/gift Plush Pepe` получает внешние данные и цены; `bot/sync_gifts.py` выгружает снапшоты. WebP/TGS-ассеты загружаются из связанного открытого CDN.

## Структура

```text
bot/main.py                 Python Telegram-бот и платёжные handlers
bot/requirements.txt        Зависимости Python
bot/Dockerfile              Отдельный Railway worker
bot/sync_gifts.py           Синхронизация TelegramGifts
src/app/                    Mini App, legal pages и API
src/components/             Кейсы, апгрейд, инвентарь, кошелёк, TGS
src/lib/catalog.ts          Подарки, кейсы и точные веса
src/lib/config.ts           Курсы, бонусы и OWNER TUNING шансов
src/lib/fairness.ts         HMAC/seed/roll
src/db/schema.ts            PostgreSQL/Drizzle
railway.json                Build/start/healthcheck web-сервиса
.env.example                Все настройки (секреты сюда не вписывать в Git)
```

## 1. Создай бота

1. Открой `@BotFather` → `/newbot`, получи токен.
2. В BotFather включи нужные команды (можно отправить список из `/help`).
3. После деплоя настрой Menu Button на URL Mini App или просто используй кнопку `/start`.
4. Узнай свой Telegram ID и добавь его в `ADMIN_IDS`.

## 2. Загрузи на GitHub

Загрузи **все файлы репозитория**, включая `bot`, `src`, `package.json`, `railway.json`, `drizzle.config.ts`. Файл `.env` не загружай: настройки задаются в Railway Variables. Для локальной настройки скопируй `.env.example` в `.env`.

## 3. Railway: база и Web

1. **New Project → Deploy from GitHub Repo** и выбери репозиторий.
2. Добавь **PostgreSQL** в этот же проект.
3. В Variables Web-сервиса задай:

```dotenv
DATABASE_URL=${{Postgres.DATABASE_URL}}
BOT_TOKEN=токен_от_BotFather
MINI_APP_URL=https://ТВОЙ-WEB-ДОМЕН.up.railway.app
API_BASE_URL=https://ТВОЙ-WEB-ДОМЕН.up.railway.app
BOT_INTERNAL_SECRET=длинная_случайная_строка_минимум_48_символов
ADMIN_IDS=6658790377
BOT_USERNAME=username_твоего_бота_без_@
REQUIRED_CHANNEL=@WoWB0X
SUPPORT_URL=https://t.me/ТВОЙ_SUPPORT
```

`MINI_APP_URL` можно точно указать после первого выданного домена и затем перезапустить. При необходимости добавь `RUB_CARD_NUMBER` и `USDT_TRC20_ADDRESS`. Не указывай реквизиты неиспользуемого способа: приложение не создаст фиктивную заявку.

4. `railway.json` уже выполняет `npm ci && npm run build`, затем `npx drizzle-kit push && npm run start`. Healthcheck: `/api/health`.
5. Игровой профиль открывается только через Telegram Mini App с подписанными initData. Демо-режима и тестовых денег нет. Админ `6658790377` может открывать платные кейсы и делать ставки без списания баланса.

## 4. Railway: Bot worker

В том же проекте добавь ещё один сервис из **того же GitHub-репозитория**:

- **Build → Dockerfile Path:** `bot/Dockerfile`
- **Deploy → Start Command:** `python bot/main.py`
- Переменные: `BOT_TOKEN`, `BOT_USERNAME`, `REQUIRED_CHANNEL`, `MINI_APP_URL`, `API_BASE_URL`, `BOT_INTERNAL_SECRET`, `ADMIN_IDS`, `SUPPORT_URL`. `DATABASE_URL` Python-боту не нужен: платежи и операторские действия идут через защищённый Web API.
- **Не включай healthcheck HTTP** для worker: это long polling, а не web-сервер.
- Не запускай две копии polling-бота одновременно.

Web и worker должны иметь одинаковые `BOT_INTERNAL_SECRET` и `API_BASE_URL`.

## 5. Проверка

1. `/start` в боте → подробное приветствие → **Открыть WoWBOX**.
2. Открой ежедневный кейс, проверь инвентарь или начисление.
3. Перед платным кейсом пополни Stars: Web создаст Invoice, worker ответит на pre-checkout и подтвердит successful payment.
4. Для RUB/USDT создай заявку, выполни реальный перевод, отправь hash/чек. Проверь перевод самостоятельно, затем в боте:

```text
/requests
/confirm_deposit UUID_ДЕПОЗИТА
/complete_withdrawal UUID_ВЫВОДА
/reject_withdrawal UUID_ВЫВОДА
```

Команды доступны только `ADMIN_IDS`. `complete_withdrawal` нажимай **после фактической передачи NFT или ручной выплаты**. Приложение не имитирует автоматическую выплату Stars: Telegram Bot API такой произвольной операции не предоставляет.

## Настройка шансов и экономики

Открой `src/lib/config.ts`, блок **OWNER TUNING**:

- `LAUNCH_GIFT_CHANCE_SCALE = 1` — активны базовые шансы: доля подарков растёт вместе с ценой кейса;
- `LAUNCH_FREE_GIFT_CHANCE_SCALE = 0.035` — низкий launch-множитель ежедневного кейса;
- `LAUNCH_UPGRADE_CHANCE_SCALE = 0.35` — низкий launch-множитель апгрейда.

Для первых 100 профилей `EARLY_DEPOSIT_BONUS_PERCENT = 100`: пользователь получает +100% к подтверждённому пополнению, что эквивалентно скидке 50% на итоговый баланс. После неудачного апгрейда +25% эквивалентно скидке 20%.

Для базовых весов поставь все три значения в `1`. Интерфейс автоматически покажет новые фактические проценты, а оставшаяся вероятность распределится между балансными исходами. **Не меняй шансы незаметно между публикацией hash и раундом.** Новые значения применяй к будущим раундам и обновляй условия.

Веса конкретных коллекций и диапазоны баланса находятся в `src/lib/catalog.ts`. Внутренние курсы, лимит первых 100 пользователей, +100% раннего бонуса, компенсации и курсы продажи/вывода — в `src/lib/config.ts`.

## Обновление подарков

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r bot/requirements.txt
python bot/sync_gifts.py
python bot/sync_gifts.py "Plush Pepe" "Artisan Brick"
```

Снапшот сохраняется в `bot/data/gifts_snapshot.json`. Каталог и внешние рыночные цены меняются; перед добавлением новой коллекции проверь `short_name` и assets. Цены из внешнего источника **не используются автоматически для шансов**: это защищает экономику от непроверенного импорта.

## Локальный запуск

```bash
npm install
cp .env.example .env
# заполни DATABASE_URL и BOT_TOKEN; открывай Mini App через Telegram
npx drizzle-kit push
npm run dev
```

Бот отдельно:

```bash
pip install -r bot/requirements.txt
python bot/main.py
```

## Важно перед публичным запуском

- Проверь местные правила для paid random rewards и требования Telegram Stars; сервис имеет пометку 18+.
- Укажи реальные реквизиты оператора и канал поддержки в legal/поддержке.
- Пополни фактический склад NFT у оператора до предложения вывода; заявка не создаёт NFT из воздуха.
- Настрой резервное копирование PostgreSQL и наблюдение за журналами.
- Добавь бота администратором @WoWB0X, иначе Telegram не позволит проверять подписку для «Точного NFT».
- Проверь `/stars 100`, deep-link `/ref` и начисление четырёх рефералов на отдельных новых аккаунтах.
