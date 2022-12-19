# OrganiseLines

A static method in MainManager that automatically inserts line breaks to fit inside a constrained width (typically a textbox or other GUI elements). This is mostly used in [SetText](../../SetText.md), but it is also used when calling the dialog via [Backtracking](../Backtracking.md).

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

The maximum width of a line. This is typically `linebreak` from SetText.

### `size`: float

The width scale of the text. This is typically sent by SetText and corresponds to [size](../../Commands/Individual%20commands/size.md).

### `fontid`: int

The [fonttype](../../fonttype.md) that will be used to render. Typically comes from SetText.

## Returns

On `Japanese`, `text` is returned with no changes. On other languages, the same string is returned, but with LF inserted at places to indicate a line change (the LF are [Line](../../Commands/Individual%20commands/Line.md) commands without parameters in case [Singlebreak](../../Commands/Individual%20commands/Singlebreak.md) is present in the input string).

## Remarks

For details of the automatic line breaking procedure, see [Organisation procedure](Organisation%20procedure.md).
