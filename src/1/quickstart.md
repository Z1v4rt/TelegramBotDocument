# Quickstart

## Bot Father

Прежде, чем начать, нужно обратиться к [@BotFather] в Telegram.
Нажать [Create a new bot](https://core.telegram.org/bots/tutorial#obtain-your-bot-token), чтобы получить новый токен бота (по сути создать новый бот) и вернуться к документации.

[![Bot Father](docs/logo-bot-father.jpg)](https://t.me/botfather)

Bot token это обязательный ключ для бота, который позволяет отправлять и получать реквесты (requests) от Bot API. <u>**Сохраните свой токен в безопасном месте и храните от посторонных глаз**</u>, он позволяет контролировать и управлять ботом. Он выглядит примерно следующим образом:

```text
1234567:4TT8bAc8GHUspu3ERYn-KGcvsvGB9u_n4ddy
```

## Первые шаги - Hello World

Теперь есть пбот, пора вдохнуть в него жизнь! 

> [!NOTE]  
> Рекомендуется использовать последнюю версию .NET, например .NET 8, но также поддерживаются старые версии .NET Framework (4.6.1+), .NET Core (2.0+) or .NET (5.0+)

Для начала создадим консольный проект.
Создаем новый консольный прокецт для нашего бота и добавляем ссылку на пакет `Telegram.Bot` проект:

```bash
dotnet new console
dotnet add package Telegram.Bot
```

The code below fetches Bot information based on its bot token by calling the Bot API [`getMe`] method. Open `Program.cs` and use the following content:

> ⚠️ Replace `YOUR_BOT_TOKEN` with your bot token obtained from [@BotFather].

```c#
using Telegram.Bot;

var bot = new TelegramBotClient("YOUR_BOT_TOKEN");
var me = await bot.GetMe();
Console.WriteLine($"Hello, World! I am user {me.Id} and my name is {me.FirstName}.");
```

Running the program gives you the following output:

```bash
dotnet run

Hello, World! I am user 1234567 and my name is Awesome Bot.
```

Great! This bot is self-aware. To make the bot react to user messages, head to the [next page].

<!-- -->

[@BotFather]: https://t.me/botfather
[`getMe`]: https://core.telegram.org/bots/api#getme
[next page]: example-bot.md
