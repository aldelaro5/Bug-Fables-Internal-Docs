# Sound

Play a sound effect music from its file path with a configurable pitch and volume with the option to loop it at the first free AudioSource or a specific one by its id.

## Syntax

(1)

````
|sound,soundname|
````

(2)

````
|sound,soundname,audiosourceid|
````

(3)

````
|sound,soundname,audiosourceid,pitch|
````

(4)

````
|sound,soundname,audiosourceid,pitch,volume|
````

(5)

````
|sound,soundname,audiosourceid,pitch,volume,loop|
````

## Parameters

### `soundname`: string

The file path of the sound effect file without extensions to play relative from the `audio/sounds` resources directory. This must be a valid sound file path or an exception will be thrown.

### `audiosourceid`: int

The audio source id used to play the sound effect out of the 15 available normally. This must be an int between -1 and 14 or an exception will be thrown. The default value is -1 which will pick the first one that isn't playing or the last one if all of them are playing.

### `pitch`: float

The pitch to play the sound effect. This must be a valid float or an exception will be thrown. The default value is 1.0 which is the normal pitch.

### `volume`: float

The volume to play the sound effect. This must be a valid float or an exception will be thrown. The default value is 1.0 which is the normal volume.

### `loop`: `true` | `false`

Whether or not to loop the sound effect. This must be a valid bool or an exception will be thrown. The default value is `false`.

## Remarks

It is possible for this command to be used in non [Dialogue mode](../Dialogue%20mode.md) which means it is possible to play bleeps sound using this command instead of letting SetText handle it with [bleep](Bleep.md) which requires [Dialogue mode](../Dialogue%20mode.md) as it only uses a single AudioSource while this command can interact with 15 of them.
