# Full Example

В предыдущем примере [previous page](example-bot.md) мы реализвали простой функционал, используя метод `bot.OnMessage`.

Теперь мы рассмотрим метод `bot.OnUpdate` и `bot.OnError`для расширения возможностей бота.

Модифицируем файл `Program.cs` как показано ниже:

```c#
using Telegram.Bot;
using Telegram.Bot.Polling;
using Telegram.Bot.Types;
using Telegram.Bot.Types.Enums;
using Telegram.Bot.Types.ReplyMarkups;

using var cts = new CancellationTokenSource();
var bot = new TelegramBotClient("YOUR_BOT_TOKEN", cancellationToken: cts.Token);
var me = await bot.GetMe();
bot.OnError += OnError;
bot.OnMessage += OnMessage;
bot.OnUpdate += OnUpdate;

Console.WriteLine($"@{me.Username} is running... Press Enter to terminate");
Console.ReadLine();
cts.Cancel(); // stop the bot

// method to handle errors in polling or in your OnMessage/OnUpdate code
async Task OnError(Exception exception, HandleErrorSource source)
{
    Console.WriteLine(exception); // just dump the exception to the console
}

// method that handle messages received by the bot:
async Task OnMessage(Message msg, UpdateType type)
{
    if (msg.Text == "/start")
    {
        await bot.SendMessage(msg.Chat, "Welcome! Pick one direction",
            replyMarkup: new InlineKeyboardButton[] { "Left", "Right" });
    }
}

// method that handle other types of updates received by the bot:
async Task OnUpdate(Update update)
{
    if (update is { CallbackQuery: { } query }) // non-null CallbackQuery
    {
        await bot.AnswerCallbackQuery(query.Id, $"You picked {query.Data}");
        await bot.SendMessage(query.Message!.Chat, $"User {query.From} clicked on {query.Data}");
    }
}
```

Запускаем программу от отправляем боту команду `/start`.
> [!NOTE]  
> `/start` эта команда является первым сообщением которое принимает бот автоматически при общении с пользователем при первом запуске

Бот пришлет приветственное сообщение и две встроенные(_inline_) кнопки для выбора.

При клике на кнопку бот принимает тип Обновления в виде **CallbackQuery** чт оне является простым сообщением.  
Поэтому обработка будет осуществляться через метод `OnUpdate`.

Мы пересылаем callback data _(which could be different from the button text)_,
and which user clicked on it _(which could be any user if the message was in a group)_

The `OnError` method handles errors, and you would typically log it to trace problems in your bot.

Look at [the Console example](https://github.com/TelegramBots/Telegram.Bot.Examples/tree/master/Console) in our [Examples repository](https://github.com/TelegramBots/Telegram.Bot.Examples) for an even more complete bot code.

<!-- -->
