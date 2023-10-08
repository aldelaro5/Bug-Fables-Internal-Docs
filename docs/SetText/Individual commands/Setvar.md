# Setvar

Set a [flagvar](../../Flags%20arrays/flagvar.md) to a specific value, or increase/decrease a [flagvar](../../Flags%20arrays/flagvar.md) by a value or another [flagvar](../../Flags%20arrays/flagvar.md).

## Syntax

(1)

````
|setvar,flagvar,value|
````

(2)

````
|setvar,operation,flagvarslot,value|
````

(3)

````
|setvar,operation,flagvarslot,var,flagvarvalue|
````

## Parameters

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot to set its value. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `operation`: `add` | `sub`

The operation to perform between the `flagvar` slot and the resolved value. `add` is equivalent to do `flagvar` = `flagvar` + value * 1 while `sub` is equivalent to do `flagvar` = `flagvar` + value * -1. Any other value will be interpreted as `flagvar` which may cause an exception to be thrown.

### `value`: int

The value used to calculate the new value to set the `flagvar` slot to. This must be a valid int or an exception will be thrown.

### `var`

The presence of this parameter indicates to operate in syntax (3) which retrieves `value` using `flagvarvalue` instead. Any other value will be interpreted as `value` which may cause an exception to be thrown.

### `flagvarvalue`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot used to retrieve the value used to calculate the new value of `flagvar`. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

Syntax (2) and (3) are a less flexible version of [addvar](Addvar.md). While the functionalities are equivalent, [addvar](Addvar.md) provides multiplication and integer division as well as the ability for the value to be the current berries amount.
