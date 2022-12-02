# Flag

Set a specific [flags](../../../Flags%20arrays/flags.md) slot or a [flags](../../../Flags%20arrays/flags.md) slot whose id is in a [flagvar](../../../Flags%20arrays/flagvar.md) to a value.

## Syntax

(1)

````
|flag,flagid,value|
````

(2)

````
|flag,dummy,flagvar,value|
````

## Parameters

### `dummy`: var

This parameter has no meaning, but its presence is required to process this command using the (2) form since the amount of parameters is what determines how the flags slot is obtained.

### `flagid`:  int

The [flags](../../../Flags%20arrays/flags.md) slot to set. This must corresponds to an integer of a valid [flags](../../../Flags%20arrays/flags.md) slot or an exception will be thrown.

### `flagvar`:  int

The [flagvar](../../../Flags%20arrays/flagvar.md) containing the id of the flags slot to set. This must corresponds to an integer of a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `value`: `true` | `false`

The value to set the [flags](../../../Flags%20arrays/flags.md) slot to. This must be equal to `true` or `false` without case sensitivity or an exception will be thrown.

## Remarks

None.
