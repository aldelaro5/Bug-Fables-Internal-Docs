# Music

Change the current music from its filename with a configurable fade speed or fade the current music to silence.

## Syntax

(1)

````
|music,musicname|
````

(2)

````
|music,musicname,fadespeed|
````

(3)

````
|music,null|
````

## Parameters

### `musicname`: string

The name of the music file without extensions to play from the `audio/music` resources directory. This must be a valid music filename or an exception will be thrown. TODO: document these?

### `fadespeed`: float

The speed to fade the current music expressed in approximate percentage to fade every frame. This must be a valid float or an exception will be thrown. The default value is 0.1.

### `null`

The presence of this value indicates to operate in syntax (3) which will fade the current music to silence, but not play another one.

## Remarks

It is not possible to configure the `fadespeed` while using syntax (3) which means the fade speed will be 0.1.
