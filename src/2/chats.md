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

Все способы общения с чатами _(например, отправлять сообщения и так далее)_ имеют параметр `ChatId`.

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
	- Фото _(использует `GetInfoAndDownloadFile` и `photo.BigFileId` чтобы загрузить)_
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
> По умолчанию, из соображений конфиденциальности, бот в группе принимает сообщения только нацеленные на него самого (адресованные боту, пересылаемые ему сообщения, вложенные сообащения  или нацеленные `/commands@botname` с именем пользотвалея бота в качестве суффикса)  
> Если нужно принимать все сообщения в группе, необходимо назначить административные права, или установить значение <u>disable</u> в **Bot Settings** : [**Group Privacy** режим](https://core.telegram.org/bots/features#privacy-mode) в [@BotFather](https://t.me/botfather)

## Миграция в Супергруппу

Когда создается приватная группа в Telegram, то это обычно тип чата `ChatType.Group`.

Если количество учасников чата превышает 200, или если изменяются некоторые настройки _(например, сделать её публичной, открыть историю сообщения для новичков или использовать пользовательские права администратора)_,
группа может мигрировать в **Супергруппу**.

В таком случае Супергруппа — это отдельный чат с другим ID. 
Старая группа будет иметь сервисное сообщение `MigrateToChatId` указывающее на новый ID Супергруппы.
Новая Супергруппа будет иметь сервисное сообщение `MigrateFromChatId` указывающее на ID старой группы.

## Управление новыми участниками в группе

Боты не могут напрямую добавлять участников группы или каналы.  
Для приглашения пользователя присоединиться к группе или каналу  можно отправить публичную ссылку-приглашение в виде `https://t.me/chatusername` (если чат имеет поле username), или пригластельную ссылку:

### Пригласительная ссылка (ссылка - приглашение)

Ссылка - приглашение обычно имеет вид `https://t.me/+AAS0mE-tH1nG` и доступна пользователям, кому она была отправлена для присоединения к группе или каналу. 

Ссылку можно отправить в текстовом сообщени используя метод `InlineKeyboardButton.WithUrl(...)`.

Если бот имеет права администратора в личных (приватных, или публичных) группах/каналах то он может:
- читать (фиксировать) основную ссылку чата:
```csharp
var chatFullInfo = await bot.GetChat(chatId); // you should call this only once
Console.WriteLine(chatFullInfo.InviteLink);
```
- создавать новую ссылку-приглашение 
```csharp
var link = await bot.CreateChatInviteLink(chatId, "name/reason", ...);
Console.WriteLine(link.InviteLink);
```

еще тут [некоторые другие методы управления ссылками на приглашения](https://core.telegram.org/bots/api#exportchatinvitelink).

### Обнаружение новых членов группы и изменения статуса члена

Простой подход к обнаружению новых присоединившихся к группе пользователей заключается в обработке события `MessageType.NewChatMembers`: в поле  `message.NewChatMembers` которе содержит массив деталей о новых пользователях.  
Для пользователей, покинувших чат устанавливается сервисное сообщение `message.LeftChatMember`.

Однако при различных обстоятельствах (большие группы, скрытые списки участников и т.д.), эти сервисные сообщения не могут быть отправлены.  

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
