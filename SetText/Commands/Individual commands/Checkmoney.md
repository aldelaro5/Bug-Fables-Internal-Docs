# Checkmoney

Check that the berry count is less than a specific value or a value contained in a [flagvar](../../../Flags%20arrays/flagvar.md). If it is, redirect to another dialogue line.

## Syntax

(1)

````
|checkmoney,value,dialogue|
````

(2)

````
|checkmoney,var,flagvar,dialogue|
````

## Parameters

### `value`:  int

The value to compare the berry count against. This must corresponds to a valid int value or an exception will be thrown.

### `var`

This is what determines how to obtain the value to compare the berry count against. Any other value will cause this to be interpreted as the `value` parameter which will throw an exception.

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the value to compare the berry count against. This must corresponds to an int of a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `dialogue`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to process if the berry count is inferior than the value to compare against. This value must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

Whenever the input string is replaced, it is done with an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of dialogue string prepended with |[blank](Blank.md)\|.

If the input string is replaced, this command will resume processing at the start of the new input string.
