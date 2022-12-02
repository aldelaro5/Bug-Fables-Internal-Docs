# Singlebreak

Instruct [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) to use the specialized line breaks handling for [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md)

## Syntax

````
|singlebreak,maxoffset|
````

## Parameters

### `maxoffset`: float

The max offset a line can grow until inserting a line break. The value must a valid `float` value or an Exception will be thrown.

## Remarks

Overwrite the input string with [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) starting with a maximum line width of 99999. The maximum line width gets overridden from the point where this command is located until the end of the input string by `maxoffset`. 

This also changes the way [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) works by changing the final line breaks scheme. Normally, [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) inserts LF for every new lines, but because this scheme isn't supported in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md), this wouldn't work. This command aims to fix this by changing the LF to [Line](Line.md) command without parameters which works in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md).

This command only works in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) which can be enabled via the [Single](Single.md) command. No operation occurs if it is not enabled.
