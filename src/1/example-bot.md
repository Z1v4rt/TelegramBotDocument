# Your First Chat Bot

В предыдущем разделе [previous page](quickstart.md) мы получили токен бота ,использовали метод [`getMe`](https://core.telegram.org/bots/api#getme) чтобы проверить функциональность нашего бота.
Настало время сделать бот _интерактивным_, то есть реагирующим на сообщения пользователелй, как на скриншоте:

![Example Image](docs/shot-example_bot.jpg)

Скопируем нижний код в файл `Program.cs`.

> ⚠️ Замените `YOUR_BOT_TOKEN` на токен бота, полученного от [@BotFather](https://t.me/botfather).

```c#
using Telegram.Bot;
using Telegram.Bot.Types;
using Telegram.Bot.Types.Enums;

using var cts = new CancellationTokenSource();
var bot = new TelegramBotClient("YOUR_BOT_TOKEN", cancellationToken: cts.Token);
var me = await bot.GetMe();
bot.OnMessage += OnMessage;

Console.WriteLine($"@{me.Username} is running... Press Enter to terminate");
Console.ReadLine();
cts.Cancel(); // stop the bot

// method that handle messages received by the bot:
async Task OnMessage(Message msg, UpdateType type)
{
    if (msg.Text is null) return;	// we only handle Text messages here
    Console.WriteLine($"Received {type} '{msg.Text}' in {msg.Chat}");
    // let's echo back received text in the chat
    await bot.SendMessage(msg.Chat, $"{msg.From} said: {msg.Text}");
}
```

Run the program:

```bash
dotnet run
```

It runs waiting for text messages unless forcefully stopped by pressing Enter. Open a private chat with your bot in
Telegram and send a text message to it. Bot should reply immediately.

By setting `bot.OnMessage`, the bot client starts polling Telegram servers for messages received by the bot.
This is done automatically in the background, so your program continue to execute and we use `Console.ReadLine()` to keep it running until you press Enter.

When user sends a message, the `OnMessage(...)` method gets invoked with the `Message` object passed as an argument (and the type of update).

We check `Message.Type` and skip the rest if it is not a text message.
Finally, we send a text message back to the same chat we got the message from.

If you take a look at the console, the program outputs the `chatId` numeric value.  
In a private chat with you, it would be your `userId`, so remember it as it's useful to send yourself messages.

```text
Received Message 'test' in Private chat with @You (123456789).
```

<!-- -->
