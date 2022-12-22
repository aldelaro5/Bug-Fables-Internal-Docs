# Resetmaplimits

Restore the position bound limits of the camera from their original one when the map was loaded or from the ones saved using [Removemaplimits](Removemaplimits.md) if they were saved to the temporary fields.

## Syntax

(1)

````
|resetmaplimits|
````

(2)

````
|resetmaplimits,resetfromtemp|
````

## Parameters

### `savetemp`: `true` | `false`

Whether or not to restore the bound limits before from the temporary fields. This must be a valid bool or an exception will be thrown. The default value is `false`.

## Remarks

If `resetfromtemp` is true, the limit bounds are restored from `tempcneg` and `tempcpos` for the lower and upper bounds respectively which are only saved using [Removemaplimits](Removemaplimits.md). Otherwise, they are restored from the values saved on the last map load.
