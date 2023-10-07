# Money

Add or remove a certain amount of berries from the berries count which can optionally come from a [flagvar](../../Flags%20arrays/flagvar.md).

## Syntax

(1)

````
|money,amount|
````

(2)

````
|money,var,flagvar|
````

(3)

````
|money,var,-,flagvar|
````

## Parameters

### `amount`: int

The amount of berries to add. This must be a valid int value or an exception will be thrown.

### `var`

This is just what determines how the amount of berries to add or remove is obtained. Any other value will this parameter to be interpreted as the `amount`.

### `-`

This indicates to remove berries instead of adding them from the amount in `flagvar`. Any other value will behave like (2). if `var` is sent accordingly. An empty value will cause an exception to be thrown.

### `flagvar`:  int

The [flagvar](../../Flags%20arrays/flagvar.md) slot containing the amount of berries to add or remove. This must corresponds to an int of a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

After the addition or subtraction has been performed, the berries count is clamped from 0 to 999.
