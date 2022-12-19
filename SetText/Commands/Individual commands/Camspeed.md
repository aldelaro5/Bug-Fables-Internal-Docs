# Camspeed

Set the camera's movement speed's multiplier.

## Syntax

````
|camspeed,speed|
````

## Parameters

### `speed`: float

The value to set the camera's speed. This must be a valid float or an exception will be thrown. Any value at 0 or below will be overridden to 0.1 which is the default speed.

## Remarks

The default speed of the camera is 0.1 while 1 would make any movement instant.
