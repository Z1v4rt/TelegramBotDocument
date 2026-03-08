# Взаимодействие с чатами

Все сообщения в Telegram отправляются и принимаются в спецефичных типах чатов.  
Тип `chat.Type` может быть четырех типов:

- `ChatType.Private`:  
  Приватня беседа с пользователем. Идентификатор `chat.Id` то же самое, что и `user.Id` (положительное число)
- `ChatType.Group`:  
  Приватная чат - группа с количеством пользователей до 200
- `ChatType.Supergroup`:  
  Продвинутая чат-группа, способная быть публичной, поддерживающая более 200 пользователей с определёнными правами пользователей/администраторов
- `ChatType.Channel`:  
  Трансляционная лента для публикации (канал, только администраторы могут писать в неё)

Дополнительные заметки:
- для групп/каналов, параметр `chat.Id` является отрицательным числом, и параметр `chat.Title` будет заполнен.
- Для  <u>public</u> (публичных) групп/каналов параметр `chat.Username` будет заполнен.
- Для приватных (личных) чатов с пользователем, параметр `chat.FirstName` будет заполнен, и, опционально, `chat.LastName` и `chat.Username` если имеются, то тоже будут заполнены.

## Вызов метода

Все способы общения с чатами All _(например, отправлять сообщения и так далее)_ имеют параметр `ChatId`.

Для этого параметра можно сразу передать число типа `long` _(идентификатор чата или пользователя)_,
или при отправке в публичную группу/канал можно передать строку `"@chatname"`.

### Получение полной информации о чате (`GetChat`)

Когда бот присоединяется к группе или каналу и начинает принимать сообщения от пользователей, он использует метод `GetChat`, чтобы получить детали о чате или ьпользователе.

В зависимости от типа чата возвращается много информации, и большинство из возвращаемых данных необязательны и могут быть недоступны. 
Вот несколько интересных примеров:
* Для личниых(приватных) чатов с пользователем:
	- Дата рождения
	- персональный канал
	- [бизнес](../4/business.md) инфо
	- биоометрия
* Для групп/каналов:
	- Описание
	- права доступа по умолчанию _(не административные права)_
	- присоединенный чат (обсуждение), его идентификатор ChatId
	- параметр IsForum _(означает что в чате есть форум (обсуждение) [темы](#forum--topics))_
* Основная информация для всех типов чатов:
	- Фотоo _(использует `GetInfoAndDownloadFile` и `photo.BigFileId` чтобы загрузить)_
	- Активные пользователи _(премиум-чаты пользователей и публичные могут иметь несколько имён пользователей)_
	- Доступные реакции для этого чата
	- Прикрепленные сообщения _(обычно самые последние)_


## Прием сообщений чата

см. главу [полчение обновлений](../3/updates) о том, как получать обновления сообщений.

Для групп и приватных чатов, принимаются обновления типа `UpdateType.Message` (имеется ввиду, что только поле `update.Message` будеть иметь значение)

Для каналов принимаются обновления с заполненным полем `update.ChannelPost`.

Для [бизнес](../4/business) сообщений, принимаются обновления с заполненным полем `update.BusinessMessage`.

Если изменятеся уже доставленное сообщение (редактируется), принимается обновление с заполненным полем `update.Edited*`

заметка: если используется метод(событие) `bot.OnMessage`, можно просто проверить аргумент UpdateType.

> [!IMPORTANT]  
> By default, for privacy reasons, bots in groups receive only messages that are targeted at them (reply to their messages, inline messages, or targeted `/commands@botname` with the bot username suffix)  
> If you want your bot to receive ALL messages in the group, you can either make it admin, or <u>disable</u> the **Bot Settings** : [**Group Privacy** mode](https://core.telegram.org/bots/features#privacy-mode) in [@BotFather](https://t.me/botfather)

## Migration to Supergroup

When you create a private chat group in Telegram, it is usually a `ChatType.Group`.

If members count reach 200, or if you change some settings _(like making it public, enabling newcomers history, or custom admin rights)_,
the group may be migrated into a **supergroup**.

In such case, the Supergroup is like a separate chat with a different ID. 
The old Group will have a service message `MigrateToChatId` with the new supergroup ID.
The new Supergroup will have a service message `MigrateFromChatId` with the old group ID.

## Managing new members in a group

Bots can't directly add members into a group/channel.  
To invite users to join a group/channel, you can send to the users the public link `https://t.me/chatusername` (if chat has a username), or invite links:

### Invite links

Invite links are typically of the form `https://t.me/+AAS0mE-tH1nG` and allow users clicking on them to join the chat.

You can send those links as a text message or as an `InlineKeyboardButton.WithUrl(...)`.

If your bot is administrator on a private (or public) group/channel, it can:
- read the (fixed) primary link of the chat:
```csharp
var chatFullInfo = await bot.GetChat(chatId); // you should call this only once
Console.WriteLine(chatFullInfo.InviteLink);
```
- create new invite links on demand
```csharp
var link = await bot.CreateChatInviteLink(chatId, "name/reason", ...);
Console.WriteLine(link.InviteLink);
```

See also [some other methods for managing invite links](https://core.telegram.org/bots/api#exportchatinvitelink).

### Detecting new group members and changed member status

The simpler approach to detecting new members joining a group is to handle service messages of type `MessageType.NewChatMembers`: the field `message.NewChatMembers` will contain an array of the new User details.  
Same for a user leaving the chat, with the `message.LeftChatMember` service message.

However, under various circumstances (bigger groups, hidden member lists, etc..), these service messages may not be sent out.  

The more complex (and more reliable) approach is instead to handle updates of type `UpdateType.ChatMember`:

* First you need to enable this specific update type among the `allowedUpdates` parameter when calling `GetUpdates`, `SetWebhook` or `StartReceiving`+`ReceiverOptions`.
* Typically, you would pass `Update.AllTypes` as the allowedUpdates parameter.
* After that, you will receive an `update.ChatMember` structure for each user changing status with their old & their new status
* The `OldChatMember`/`NewChatMember` status fields can be one of the derived `ChatMember*` class: `Owner`/`Creator`, `Administrator`, `Member`, `Restricted`, `Left`, `Banned`/`Kicked`)

### Forum & Topics

Group owners can enable the **Forum** feature on their chat, which allows them to create **[topics](https://telegram.org/blog/topics-in-groups-collectible-usernames#topics-in-groups)** for specialized discussions.

Messages in topics are indicated with the `MessageThreadId` property (or the `messageThreadId:` argument when sending). This value is equal to 1 for the **General** topic, or to the Message ID of the first message in topic.

Bots can [create/edit/close/reopen/delete](https://core.telegram.org/bots/api#createforumtopic) specific topics or the General topic.

**Important:** Bots can't fetch the current list of all topics in the chat.  
However, your bot can keep track of active topics by listening to these service messages (with `MessageThreadId` set):
- `MessageType.ForumTopicCreated` and the `message.ForumTopicCreated` structure
- `MessageType.ForumTopicEdited` and the `message.ForumTopicEdited` structure
- `MessageType.ForumTopicClosed`
- `MessageType.ForumTopicReopened`
- ...
