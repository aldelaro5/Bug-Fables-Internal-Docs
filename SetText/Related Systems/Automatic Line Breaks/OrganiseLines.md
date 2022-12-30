# OrganiseLines

A static method in MainManager that automatically inserts line breaks to fit inside a constrained width (typically the [textbox](../../Notable%20local%20variable/textbox.md) or other GUI elements). This is mostly used in [SetText](../../SetText.md), but it is also used when calling the dialog via [Backtracking](../Backtracking.md).

This method works on all languages except `Japanese` where it will immediately return `text`.

`English` works slightly differently as the line width calculations are broken (see [OrganiseLines Known Issues > Not counting a whole word's width after the first line](OrganiseLines%20Known%20Issues.md#not-counting-a-whole-word-s-width-after-the-first-line) for more details).

## Signature

````cs
private static string OrganizeLines(string text, float maxoffset, float size, int fontid)
````

## Parameters

### `text`: string

The text to have the line breaks inserted.

### `maxoffset`: float

The maximum width of a line. This is usually the linebreak's parameter sent to [SetText](../../SetText.md), but it can be overridden by [Setbreak](../../Commands/Individual%20commands/Setbreak.md) (doing so is not recommended).

### `size`: float

The horizontal scale of the text. This is usually sent by [SetText](../../SetText.md) and corresponds to the current [size](../../Commands/Individual%20commands/size.md).

### `fontid`: int

The [fonttype](../../fonttype.md) that will be used to render. Typically comes from [SetText](../../SetText.md) using the current one.

## Returns

On `Japanese`, `text` is returned with no changes. On other languages, the same string is returned, but with LF inserted at places to indicate a line change (the LF are [Line](../../Commands/Individual%20commands/Line.md) commands without parameters in case [Singlebreak](../../Commands/Individual%20commands/Singlebreak.md) is present in the input string).

## Remarks

There are a couple of caveats to this method because of its design. The main one is it is not able to "look ahead" of the input string so for example, it cannot know if there will be redirection or text replacement performed during processing. While there are some specific cases where this method actually processes commands (such as [Sstring](../../Commands/Individual%20commands/Sstring.md) and [Menu](../../Commands/Individual%20commands/Menu.md)), they are far from exhaustive.

Additionally, there are some known issues that are important to mention which you can learn more at [OrganiseLines Known Issues](OrganiseLines%20Known%20Issues.md).

For details of the automatic line breaking procedure, see [Organisation procedure](Organisation%20procedure.md).
