# Addvar

Perform an arithmetic operation on a [flagvar](../../Flags%20arrays/flagvar.md) slot using a value as the other operand where the value is directly specified, corresponds to the current amount of berries or contained in another [flagvar](../../Flags%20arrays/flagvar.md) slot.

## Syntax

(1)

````
|addvar,flagvar,value|
````

(2)

````
|addvar,flagvar,value,operation|
````

## Parameters

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot to set its value. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `value`: int | `var`int | `v`int | `money`

The value to set `flagvar` [flagvar](../../Flags%20arrays/flagvar.md) slot to. The int form must be a valid int or an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. The `money` value corresponds to the current amount of berries.

### `operation`: `-` | `*` | `/`

The operation to perform between the `flagvar` slot and the resolved value. `-` is equivalent to do `flagvar` = `flagvar` + value * -1, `*` is equivalent to do `flagvar` = `flagvar` * value and `/` is equivalent to do `flagvar` = `flagvar` / value (this is an integer division which means any decimal part is truncated and it act as a floor). The default is to perform an addition or `flagvar` = `flagvar` + value. Any other value will be ignored and an addition will be performed instead.

## Remarks

This is essentially a more flexible version of syntax (2) and (3) of [setvar](Setvar.md), but other than that, functionality is very similar. The main benefit of using this command over [setvar](Setvar.md) is it allows multiplication and integer division as well as the value being the current berries amount. [setvar](Setvar.md) or [copyvar](Copyvar.md) should still be used to set a [flagvar](../../Flags%20arrays/flagvar.md) to a specific value however.
