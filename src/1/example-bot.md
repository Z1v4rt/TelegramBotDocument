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

Запустим программу:

```bash
dotnet run
```

Бот запустится в режиме ожидания сообщения от пользователя, чтобы остановить бот надо нажать Enter. Начните приватный чат с ботом в Telegram и отправьте текстовое сообщение ему.Бот должен ответить немедленно.

Метод `bot.OnMessage` запускает клиента опроса серверов Telegram для приема новых сообщений от бота.
Это выполняется автоматичести в фоновом процессе, пока программа продолжает выполнение и метод `Console.ReadLine()` ждет нажатия клавиши Enter.

Когда пользователь отправляет сообщение , метод `OnMessage(...)` получет вызов объекта `Message` в качестве аргумента (и тип обновления).

Мы проверяем тип сообщения `Message.Type` и пропускаем все, что не является текстом.
В конце пересылается текст принятого сообщения обратно в чат, откуда оно было прислано. 

В консоли при этом будет выведено числовое значение `chatId`.  
В личном чате это будет `userId` того, кто отправил это сообщение боту.

```text
Received Message 'test' in Private chat with @You (123456789).
```

<!-- -->
