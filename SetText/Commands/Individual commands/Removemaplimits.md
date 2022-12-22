# Removemaplimits

Remove the position bound limits of the camera with the option to save the current ones to restore later using [Resetmaplimits](Resetmaplimits.md).

## Syntax

(1)

````
|removemaplimits|
````

(2)

````
|removemaplimits,savetemp|
````

## Parameters

### `savetemp`: `true` | `false`

Whether or not to save the current bound limits before removing them to temporary fields. This must be a valid bool or an exception will be thrown. The default value is `false`.

## Remarks

If `savetemp` is true, the current limits are saved to `tempcneg` and `tempcpos` for the lower and upper bounds respectively. They can be restored later using [Resetmaplimits](Resetmaplimits.md).
