# Shakecamera

Shakes the camera's position by a configurable amount, time and whether or not to return to the initial position.

## Syntax

(1)

````
|shakecamera,amount,time|
````

(2)

````
|shakecamera,amount,time,true|
````

## Parameters

### `amount`: float

The amount of maximum units to displace the camera horizontally and half the maximum amount vertically. This must be a valid float or an exception will be thrown. This essentially allows to move the camera from (-amount, -(amount / 2)) to (amount, amount / 2). every FixedUpdate.

### `time`: float

The amount of time in seconds before stopping the shaking. This must be a valid float or an exception will be thrown.

### `true`

The presence of this parameter indicates to not reset the camera's position to its initial one after the shake. Any other value or its absence indicates to reset the position after the shake.
