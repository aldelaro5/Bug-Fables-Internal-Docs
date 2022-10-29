A static method in MainManager. This is mostly used in SetText, but it is also used when calling the dialog via [Backtracking](../Related%20Systems/Backtracking.md).

This is a very complex method that seems to be integrated to SetText in an odd way as its primary goal seems to do some processing of the input string to replace it to an organized version. This is potentially related to line breaks??? Oddly, it does nothing to the input string if the [languageid](../languageid.md) is `Japanese`.

## Signature

````cs
private static string OrganizeLines(string text, float maxoffset, float size, int fontid)
````

## Parameters

### text

The text to have the lines organised.

### maxoffset

The maximum width of a line. This is typically `linebreak` from SetText.

### size

The size the text is scaled at. This is typically sent by SetText.

### fontid

The [fonttype](../fonttype.md) that will be used to render. Typically comes from SetText.

## Returns

An organised version of the input string?
