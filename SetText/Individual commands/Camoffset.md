# Camoffset

Changes the main camera's base position.

## Syntax

````
|camoffset,x,y,z|
````

## Parameters

### `x`: float

The x base position to set. This must be a valid float or an exception will be thrown.

### `y`: float

The y base position to set. This must be a valid float or an exception will be thrown.

### `z`: float

The z base position to set. This must be a valid float or an exception will be thrown.

## Remarks

This doesn't entirely change the camera's position: it only sets the `camoffset` field while the actual position is `camoffset` + `camoffset2`. The later is by default Vector3.zero, but it is possible to change its value from code.
