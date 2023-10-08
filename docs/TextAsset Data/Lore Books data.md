# Lore Books data

The game contains a TextAsset named `LoreText` in the corresponding dialogue folder of the current [languageid](../SetText/languageid.md) and it contains information about the Lore Books. The data is loaded each time by the [Lore Book List Type](../ItemList/List%20Types%20Group%20Details/Lore%20Book%20List%20Type.md) and the [Lore](../SetText/Individual%20commands/Lore.md) command's processing.

The asset contains the list of Lore Books, one per line, in the order they are unlocked as the player turns in Lore Books to Brooke. Each lines contains fields separated by `@`:

|Name|Type|Description|
|----|----|-----------|
|Title|[SetText](../SetText/SetText.md) string|The title of the book shown in the [Lore Book List Type](../ItemList/List%20Types%20Group%20Details/Lore%20Book%20List%20Type.md)|
|Content|[SetText](../SetText/SetText.md) string|The content of the book's text.|

The content of the book is expected to be rendered using a [Lore](../SetText/Individual%20commands/Lore.md) command which happens through the handling of the [Lore Book List Type](../ItemList/List%20Types%20Group%20Details/Lore%20Book%20List%20Type.md).
