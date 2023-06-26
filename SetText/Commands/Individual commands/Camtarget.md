# Camtarget

Set the camera to target an [Entity](../../../Entities/Entity.md), target a specific position or remove any existing targets.

## Syntax

(1)

````
|camtarget,target|
````

(1)

````
|camtarget,null|
````

(2)

````
|camtarget,x,y,z|
````

## Parameters

### `target`: int | string

The [Entity](../../../Entities/Entity.md) to have the camera targets. The int form is a regular [Entity id](../Entity%20id.md) and its restriction for valid values applies. The string form can be one of the following:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller's entity. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [Define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `null`

Indicate to remove any existing camera targets. Any other value will be interpreted as `target`.

### `x`: float

The x position for the camera to target. This must be a valid float or an exception will be thrown.

### `y`: float

The y position for the camera to target. This must be a valid float or an exception will be thrown.

### `z`: float

The z position for the camera to target. This must be a valid float or an exception will be thrown.

## Remarks

None.
