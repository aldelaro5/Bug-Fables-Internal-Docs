# Singlebreak

Instruct [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) to use the specialized line breaks handling for [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)

## Syntax

````
|singlebreak,maxoffset|
````

## Parameters

### `maxoffset`: float

The max offset a line can grow until inserting a line break. The value must a valid float value or an Exception will be thrown.

## Remarks

Overwrite the input string with [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) starting with a maximum line width of 99999. The maximum line width gets overridden from the point where this command is located until the end of the input string by `maxoffset`. 

This also changes the way [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) works by changing the final line breaks scheme and by changing how letter widths are accumulated (0.975x of each letter's width is accumulated instead of their full width). Normally, [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) inserts LF for every new lines, but because this scheme isn't supported in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), this wouldn't work. This command aims to fix this by changing the LF to [line](Line.md) command without parameters which works in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).

This command only works in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) which can be enabled via the [single](Single.md) command. No operation occurs if it is not enabled, however, if [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) is ran on the input string through any ways, it will still change its logic because it is processed there no matter the rendering method. This means that this command can change the line breaker logic without necessarily using [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).
