# String

Replaces the text from this command to a [flagstring](../../../Flags%20arrays/flagstring.md) text with horizontal size clamping support.

## Syntax

(1)

````
|string,flagstring|
````

(2)

````
|string,flagstring,clamp,maxwidth|
````

(3)

````
|string,flagstring,clamp,maxwidth,widthscaleclamp|
````

(4)

````
|string,flagstring,true|
````

## Parameters

### `flagstring`:  int

The [flagstring](../../../Flags%20arrays/flagstring.md) slot to replace this command with. This must be a valid [flagstring](../../../Flags%20arrays/flagstring.md) slot or an exception will be thrown.

### `postprocessing`: `clamp` | `true`

Specify the optional post processing step to perform after obtaining the [flagstring](../../../Flags%20arrays/flagstring.md) text (this is case sensitive):

* `clamp`: Before replacing the text of this command, Clamps the horizontal size of the [flagstring](../../../Flags%20arrays/flagstring.md) text if it is wider than `maxwidth` .
* `true`: After the text replacement, reorganize the lines of the input string via [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) just like in the SetText [SetText Life Cycle > Setup](../../SetText%20Life%20Cycle.md#setup)

Any other value of this parameter will be ignored and the behavior will be like (1). If the value is `clamp`, but no `maxwidth` is specified, an exception will be thrown.

### `maxwidth`: float

When `postprocessing` is `clamp`, specify the maximum length allowed for the [flagstring](../../../Flags%20arrays/flagstring.md) text to take horizontally. The value must a valid float or an Exception will be thrown.

### `widthscaleclamp`: float

When `postprocessing` is `clamp`, specify the horizontal scale to apply to the [flagstring](../../../Flags%20arrays/flagstring.md) text if its width exceeds `maxwidth`. If this value is not specified, the default is 0.5. The value must a valid float or an Exception will be thrown.

## Remarks

The [flagstring](../../../Flags%20arrays/flagstring.md) is expected to be set in code beforehand and its line endings will be normalized to LF if any CRLF is in the [flagstring](../../../Flags%20arrays/flagstring.md) text.

If `postprocessing` is `clamp` and a clamping occurs, the [flagstring](../../../Flags%20arrays/flagstring.md) text will be prepended with |[Sizemulti](Sizemulti.md),`widthscaleclamp`,1| and appended with |[size](size.md),size.x,size.y| where [size](size.md) is the parameter sent to SetText. This resizes the text using `widhscaleclamp`, but turns the [size](size.md) value back to its normal one.

This command will cause SetText to resume processing at the same character position to accommodate the text replacement.
