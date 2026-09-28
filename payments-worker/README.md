# VKMuzPremium — сервер Telegram Stars

Этот отдельный Cloudflare Worker создаёт счета Telegram Stars (`XTR`) для PRO ⭐99 и PREMIUM ⭐299. Подписка автоматически продлевается каждые 30 дней. Доступ включается только после подтверждения Telegram через `successful_payment`. Заказы, платежи и статусы хранятся в D1. Команда `/cancel` отключает автопродление, но оплаченный срок сохраняется.

Музыкальная лаборатория и аудиоэффекты Mini App — демонстрационные функции в браузере. Настоящие Stars-платежи заработают только после настройки секрета бота, подключения webhook и публикации Mini App.

## Развёртывание Worker и D1

1. Установите Node.js и Git, затем выполните:

   ```bash
   git clone https://github.com/Husniddin2006yil-cell/vkmuzpremium.git
   cd vkmuzpremium/payments-worker
   npx wrangler login
   ```

2. Для Cloudflare-аккаунта этого проекта база `vkmuzpremium_stars` уже создана, а её ID указан в `wrangler.toml`. Если вы развёртываете проект в другом аккаунте, создайте там свою базу:

   ```bash
   npx wrangler d1 create vkmuzpremium_stars
   ```

   Затем замените `database_id` в `wrangler.toml` на новый ID.

3. Создайте таблицы в удалённой D1-базе:

   ```bash
   npx wrangler d1 execute vkmuzpremium_stars --remote --file=./schema.sql
   ```

   Используйте `--remote`, чтобы команда выполнялась в Cloudflare, а не в локальной базе.

4. Добавьте **новый, действующий токен бота** как Cloudflare Worker Secret. Не помещайте его в GitHub, исходный код или публичную переменную:

   ```bash
   npx wrangler secret put BOT_TOKEN
   ```

   Введите токен в запросе Wrangler. Либо откройте Cloudflare Dashboard → Workers & Pages → `vkmuzpremium-stars-payments` → Settings → Variables and Secrets → Add → **Secret**, имя `BOT_TOKEN`. Локальный `.env` сам по себе в Cloudflare не загружается.

5. Опубликуйте Worker:

   ```bash
   npx wrangler deploy
   ```

   Ожидаемый адрес: `https://vkmuzpremium-stars-payments.husniddin2006yil.workers.dev`. Используйте URL, который Wrangler действительно покажет после публикации.

6. Проверьте `https://YOUR-WORKER-URL/health`. `configured: true` означает, что Worker видит и D1, и секрет `BOT_TOKEN`.

7. Только после этого откройте `https://YOUR-WORKER-URL/setup`, проверьте текущий webhook и ожидающие обновления. Подключение заменит прежний webhook бота; старый сервер и его функции, включая поиск музыки, могут перестать работать. Не переключайте webhook, пока не будете готовы к этому. Ожидающие обновления не удаляются.

## Mini App и Cloudflare Pages

Для статического сайта в Cloudflare Pages выберите Framework preset `None`, Build command `exit 0`, Build output directory `.`, Production branch `main`; Root directory оставьте пустым. Убедитесь, что `PAYMENTS_API_URL` в `../app.js` совпадает с фактическим URL Worker, затем укажите URL Pages как адрес Mini App в BotFather. Счёт Telegram Stars открывается внутри Telegram.

## Безопасность и тестирование

- Сначала тестируйте через тестового бота и тестовую среду Telegram. На рабочем боте списываются настоящие Stars.
- Если токен уже отправлялся в чат или публиковался, отзовите его через BotFather (`/revoke`) и создайте новый перед добавлением в Cloudflare.
- Команда `/health` проверяет доступность Worker и его конфигурацию; `getMe` проверяет токен и имя бота, но не проводит платёж.
- Не активируйте webhook, не проверив его текущий адрес и не оценив влияние на старый сервер бота.

## Маршруты Worker

- `GET /health` — состояние Worker и конфигурации.
- `POST /api/create-invoice` — создание счёта после проверки подписи Telegram `initData`.
- `POST /api/status` — получение статуса подписки.
- `POST /api/cancel` — отключение автопродления.
- `POST /telegram/webhook` — обработка `pre_checkout_query`, платежей и команд бота.
- `GET /setup` — защищённая страница проверки и подключения webhook.
- `GET /terms` — условия подписки.


## Первая сборка после подключения GitHub

Если GitHub-репозиторий подключили после последнего коммита, нажмите Retry для последней сборки в Cloudflare или отправьте новый коммит в ветку `main`, чтобы запустить первую сборку. После успешного деплоя `GET /health` должен возвращать JSON, а не `Hello world`.
