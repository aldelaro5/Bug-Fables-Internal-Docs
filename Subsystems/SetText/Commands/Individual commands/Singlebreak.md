# Singlebreak

Specialised line breaks handling in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) TODO: why?

## Syntax

````
|singlebreak,maxoffset|
````

## Parameters

### `maxoffset`: float

The max offset a line can grow until inserting a line break. The value must a valid `float` value or an Exception will be thrown.

## Remarks

Overwrite the input string with [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) starting with a maximum line width of 99999. The maximum line width gets overridden from the point where this command is located until the end of the input string by `maxoffset`. This also changes the way [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) works TODO: how does it change it?

This command only works in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) which can be enabled via the [Single](Single.md) command. No operation occurs if it is not enabled.
