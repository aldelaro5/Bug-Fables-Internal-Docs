# Exp

Set, add or remove Exploration Points to the party by a value or by a value stored in a [flagvar](../../Flags%20arrays/flagvar.md) slot.

## Syntax

(1)

````
|exp,value|
````

(2)

````
|exp,flagvar|
````

(3)

````
|exp,operation,value|
````

(4)

````
|exp,operation,flagvar|
````

## Parameters

### `value`: int

The amount of Exploration Points to set/add/remove. This must be a valid int or an exception will be thrown.

### `flagvar`: `var`int

The [flagvar](../../Flags%20arrays/flagvar.md) slot to get the value of Exploration Points to set/add/remove. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. This must contains `var` once otherwise the value will be interpreted as `value` which may cause an exception to be thrown.

### `operation`: `add` | `sub`

The operation to perform using the current amount of Exploration Points and the resolved value. `add` will add the value to the current amount while `sub` will subtract the value to the current amount. The new value is clamped from 0 to the required amount for the next level - 1 after the operation. Any other value will be interpreted as syntax `value`/`flagvar` which may cause an exception to be thrown.

## Remarks

This command only manages the actual Exploration Points counter, it does not manage the rank. For that one, use [level](Level.md).
