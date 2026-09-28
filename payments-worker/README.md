# VKMuzPremium — сервер Telegram Stars

Этот отдельный Cloudflare Worker создаёт счета Telegram Stars (`XTR`) для PRO ⭐99 и PREMIUM ⭐299. Подписка автоматически продлевается каждые 30 дней. Доступ включается только после подтверждения Telegram через `successful_payment`. Заказы, платежи и статус подписки хранятся в D1. Команда `/cancel` отключает автопродление, но оплаченный срок сохраняется.

Музыкальная лаборатория и аудиоэффекты Mini App — демонстрационные функции в браузере. Платёж проходит через настоящий поток Telegram Stars только после развёртывания Worker, подключения webhook и настройки адреса Mini App.

## Развёртывание Cloudflare Worker и D1

1. На компьютере с Node.js склонируйте репозиторий и откройте папку Worker:

   ```bash
   git clone https://github.com/Husniddin2006yil-cell/vkmuzpremium.git
   cd vkmuzpremium/payments-worker
   npx wrangler login
   ```

2. Создайте базу D1:

   ```bash
   npx wrangler d1 create vkmuzpremium_stars
   ```

   Замените `REPLACE_WITH_D1_DATABASE_ID` в `wrangler.toml` на выданный `database_id`.

3. Загрузите схему в удалённую базу D1:

   ```bash
   npx wrangler d1 execute vkmuzpremium_stars --remote --file=schema.sql
   ```

4. Добавьте токен BotFather как **секрет Worker** — не коммитьте его в GitHub:

   ```bash
   npx wrangler secret put BOT_TOKEN
   ```

   Введите токен в появившемся запросе. Либо откройте Cloudflare Dashboard → Workers & Pages → `vkmuzpremium-stars-payments` → Settings → Variables and Secrets → **Add secret**, имя `BOT_TOKEN`. Локальный `.env` сам по себе не загружается в Cloudflare.

5. Разверните Worker:

   ```bash
   npx wrangler deploy
   ```

   Ожидаемый адрес: `https://vkmuzpremium-stars-payments.husniddin2006yil.workers.dev`. Если Cloudflare выдаст другой адрес, измените `PAYMENTS_API_URL` в `../app.js` и повторно разверните сайт.

6. Откройте `https://vkmuzpremium-stars-payments.husniddin2006yil.workers.dev/setup`. В защищённой HTTPS-форме укажите токен, сначала проверьте текущий webhook и число ожидающих обновлений. Нажимайте кнопку подключения только после того, как проверили старый адрес и готовы переключить бота на новый сервер. Это заменит поток обновлений webhook; прежний сервер и функции бота могут перестать работать. Ожидающие обновления не удаляются.

7. Разверните файлы сайта через Cloudflare Pages (или текущий хостинг) и внесите адрес сайта в настройки Mini App через BotFather. Настоящий Stars-счёт открывается внутри Telegram.

## Проверка и безопасность

- Сначала тестируйте через официальный тестовый режим Telegram и тестового бота. На рабочем боте списываются настоящие Stars. Вызов `getMe` проверяет токен и имя бота, но не проверяет платёж.
- Не помещайте токен в публичный GitHub, JavaScript сайта, журналы или открытые Cloudflare-переменные. Используйте Cloudflare Worker **Secret**.
- Файл `.env` исключён из Git; `.env.example` не содержит секрета. Если токен уже был опубликован в чате или другом месте, отзовите его через BotFather (`/revoke`) и создайте новый.
- Перед переключением webhook проверьте его текущий адрес через `/setup` и убедитесь, что готовы к возможной остановке старого обработчика.

## Адреса Worker

- `GET /health` — общая проверка Worker и настроек.
- `POST /api/create-invoice` — создание счёта после проверки подписи Telegram `initData`.
- `POST /api/status` — получение статуса подписки.
- `POST /api/cancel` — отключение автопродления.
- `POST /telegram/webhook` — обработка `pre_checkout_query`, платежей и команд бота.
- `GET /setup` — просмотр и подключение webhook после подтверждения владельцем.
- `GET /terms` — условия подписки и поддержка.
