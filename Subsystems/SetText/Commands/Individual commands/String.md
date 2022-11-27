# String

Replaces the text from this command to a flagstring text with horizontal size clamping and [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) support.

## Syntax

(1)

````
|string,flagstring|
````

(2)

````
|single,flagstring,clamp,maxwidth|
````

(3)

````
|single,flagstring,clamp,maxwidth,widthscaleclamp|
````

(3)

````
|single,flagstring,true|
````

## Parameters

### `flagstring`:  int

The flagstring slot to replace this command with. This must corresponds to an integer of a valid flagstring slot or an exception will be thrown.

### `postprocessing`: `clamp` | `true`

Specify the optional post processing step to perform after obtaining the flagstring text (this is case sensitive):

* `clamp`: Before replacing the text of this command, Clamps the horizontal size of the flagstring text if it is wider than `maxwidth` .
* `true`: After the text replacement, reorganize the lines of the input string via [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) just like in the SetText [life cycle > Setup](../../life%20cycle.md#setup)

Any other value of this parameter will be ignored and the behavior will be like (1). If the value is `clamp`, but no `maxwidth` is specified, an exception will be thrown.

### `maxwidth`: float

When `postprocessing` is `clamp`, specify the maximum length allowed for the flagstring text to take horizontally. The value must a valid `float` value or an Exception will be thrown.

### `widthscaleclamp`: float

When `postprocessing` is `clamp`, specify the horizontal scale to apply to the flagstring text if its width exceeds `maxwidth`. If this value is not specified, the default is 0.5. The value must a valid `float` value or an Exception will be thrown.

## Remarks

The flagstring is expected to be set in code beforehand and its line endings will be normallized to LF if any CRLF is in the flagstring text.

If `postprocessing` is `clamp` and a clamping occurs, the flagstring text will be prepended with `|sizemulti,widthscaleclamp,1|` and appended with `|size,size.x,size.y|` where `Size` is the parameter sent to SetText.

This command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
