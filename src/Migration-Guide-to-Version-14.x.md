# Migration guide for version 14.x

## Date and Time

All `DateTime` values are now in UTC format. Here are some examples of usage:

```csharp
// Use UTC time when making a request
await BotClient.KickChatMemberAsync(
  chatId: -9876,
  userId: 1234,
  untilDate: DateTime.UtcNow.AddDays(2)
);
```

```csharp
// Convert to local time (not recommended though)
DateTime localTime = update.Message.Date.ToLocalTime();
```

## Keyboard Buttons

Many keyboard button types are removed from project. It is more convenient to use factory methods on `KeyboardButton` and `InlineKeyboardButton` classes.

Here are some examples:

```csharp
// Message having an inline keyboard button with URL that redirects to a page
await BotClient.SendTextMessageAsync(
  chatId: -9876,
  text: "Check out the source code",
  replyMarkup: new InlineKeyboardMarkup(
    InlineKeyboardButton.WithUrl("Repository", "https://github.com/TelegramBots/Telegram.Bot")
  )
);
```

```csharp
// Message to a private chat having a 2-row reply keyboard
await BotClient.SendTextMessageAsync(
  chatId: 1234,
  text: "Share your contact & location",
  replyMarkup: new ReplyKeyboardMarkup(
    new [] { KeyboardButton.WithRequestContact("Share Contact") },
    new [] { KeyboardButton.WithRequestLocation("Share Location") },
  )
);
```

## `GetFileAsync()`

Downloading a file from Telegram Bot API has 2 steps ([see docs here](https://core.telegram.org/bots/api#getfile)):

1. Get file info by calling `getFile`
1. Download file from `https://api.telegram.org/file/bot<token>/<file_path>`

`GetFileAsync()` is replaced by 3 methods. Method `GetInfoAndDownloadFileAsync()` looks very similar to old `GetFileAsync()`:

```csharp
// Gets file info and saves it to "path/to/file.pdf"
using (var fileStream = System.IO.File.OpenWrite("path/to/file.pdf"))
{
  File fileInfo = await BotClient.GetInfoAndDownloadFileAsync(
    fileId: "BsdfgLg4Khdlsn-bldBD",
    destination: fileStream
  );
}
```

> Note that calling the method `GetInfoAndDownloadFileAsync()` results in 2 HTTP requests (steps 1 and 2 above) being sent to the Bot API.

There are two more methods that assist you with downloading files:

```csharp
// New version of GetFileAsync() only gets the file info (step 1)
File fileInfo = await BotClient.GetFileAsync("BsdfgLg4Khdlsn-bldBD");

// Download file from server (step 2)
using (var fileStream = System.IO.File.OpenWrite("path/to/file.pdf")) {
  await BotClient.DownloadFileAsync(
    filePath: fileInfo.FilePath,
    destination: fileStream
  );
}
```

## `GetUpdatesAsync()`, `SetWebhookAsync()`

Value `All` is removed from enum `Telegram.Bot.Types.Enums.UpdateType`. In order to get all kind of updates, pass an empty list such as `Array.Empty<UpdateType>()` for `allowedUpdates` argument.

## `SetWebhookAsync()`

Parameter `url` is required. If you intend to remove the webhook, it is recommended to use `DeleteWebhookAsync()` instead. However, you could achieve the same result by passing `string.Empty` value to `url` argument.

## `AnswerInlineQueryAsync()` and `InlineQueryResult`

Classes `InlineQueryResultNew` and `InlineQueryResultCache` are removed. `InlineQueryResult` has become the only shared base type for all inline query result classes.

Many shared and redundant properties are removed. This might require significant changes to your `.cs` files if your bot is in _inline mode_. Fortunately, all input query results now have constructors with only the required properties as their parameters. This is the preferred way to instantiate input query result instances e.g.:

Instead of:

```csharp
// bad way. easy to get exceptions
var documentResult = new InlineQueryResultDocument
{
  Id = "some-id",
  Url = "https://example.com/document.pdf",
  Title = "Some title",
  MimeType = "application/pdf"
};
```

You should use:

```csharp
// good way
var documentResult = new InlineQueryResultDocument(
  id: "some-id",
  documentUrl: "https://example.com/document.pdf",
  title: "Some title",
  mimeType: "application/pdf"
);
```

## `SendMediaGroupAsync()`

`InputMediaType` is renamed to `InputMedia`.

> *ToDo*

## Inline Message Overloads

Many inline message methods have been replaced with their overloads.

- `EditInlineMessageTextAsync`--> `EditMessageTextAsync`

> *ToDo*

## `FileToSend`

New classes have replaced `FileToSend` struct.

- `InputFileStream`:
- `InputTelegramFile`:
- `InputOnlineFile`:

In many cases, you can use implicit casting to pass parameters.

```csharp
Stream stream = System.IO.File.OpenRead("photo.png");
var message = await BotClient.SendPhotoAsync("chat id", stream);

string fileId = "file_id on Telegram servers";
var message = await BotClient.SendPhotoAsync("chat id", fileId);
```

> *ToDo*. implicit casts

## `UpdateType` and `MessageType`

Values in these two enums are renamed e.g. `UpdateType.MessageUpdate` is now `UpdateType.Message`.

`MessageType.Service` is removed. Now each type of message has its own `MessageType` value e.g. when a chat member leaves a group, corresponding update contains a message type of `MessageType.ChatMemberLeft` value.

## `VideoNote`

Properties `Width` and `Height` are removed. Vide notes are squared and `Length` property represents both width and height.

## Constructor Parameters Instead of Public Setters

Many types now have the required parameters in their constructors. To avoid running into problems or getting exceptions, we recommend providing all required values in the constructor e.g.:

```c#
//bad way:
var markup = new InlineKeyboardMarkup
  {
    Keyboard = buttonsArray,
    ResizeKeyboard = true
  };

// better:
var markup = new InlineKeyboardMarkup(buttonsArray)
  {
    ResizeKeyboard = true
  };
```
