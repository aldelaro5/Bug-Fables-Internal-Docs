# Bleep

Change the current dialogue bleep sound and its pitch in [Dialogue mode](../../Dialogue%20mode.md) from an [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) data, [Entity](../../../Data%20format/Entity.md) field or specified directly.

## Syntax

(1)

````
|bleep,animid|
````

(2)

````
|bleep,entity|
````

(3)

````
|bleep,sound,pitch|
````

(4)

````
|bleep,sound,pitch,volume|
````

## Parameters

### `animid`: int

The [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) to get its bleep and pitch as the new dialogue bleep sound. This must be a valid [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) or an exception will be thrown. While the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) data are using the enum format to index them, this parameter assumes the int format as it will convert it before indexation.

### `entity`: `@`int | `@this`, `@caller`, `@`string)

The [Entity id](../Entity%20id.md) or designator to get its bleep and pitch as the new dialogue bleep sound. The `@` indicates to use syntax (2) as not providing it will use syntax (1) which will have this parameter be interpreted as `animid` which may cause an exception to be thrown. The int form represents an [Entity id](../Entity%20id.md) and it must be a valid int or an exception will be thrown. For other values:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md).
* `caller`: Refers to the caller.
* Anything else: Refers to a `Define` if it exists, otherwise, this is interpreted as a regular [Entity id](../Entity%20id.md) which will cause an exception to be thrown.

The string form still must resolve to a non null entity or an exception will be thrown.

### `sound`: string

The new bleep sound id to set the new dialogue bleep sound. This must be a valid bleep sound id or an exception will be thrown. This value is essentially the filename of the dialogue bleep sound without extension and without the `Dialogue` at the beginning. The files are located at `Audio/Sounds/Dialogue` from the project's resources directory.  This gives the following valid values:
Value | Description
--------- | ---------
0 | A mid range pitch, brief and very fast sound
1 | A high pitch, brief and fast sound
2 | A typewriter sound
3 | A high pitch, long and slow sound
3old | A louder version of 3
4 | A robotic sound
5 | A low pitch and slow sound
6 | A higher pitch, brief and fast sound
7 | A low pitch, long and fast sound
8 | A very low pitch, long and fast sound
9 | A very high pitch, brief and fast sound
10 | A very high pitch, brief and slow sound
11 | A high pitch, brief and slow sound
12 | A high pitch, slow and distorted sound
13 | A low pitch, long and slow sound
14 | A mid range pitch, brief and fast sound
15 | A very low pitch, long and slow sound
16 | A very low pitch, brief and very slow sound

Any other value will cause an exception to be thrown.

### `pitch`: float

The pitch value to set the dialogue bleep. This must be a valid float or an exception will be thrown. The default value at the start of a SetText call is 1.0 which is the normal pitch of the sound.

### `volume`: float

The pitch volume to set the dialogue bleep. This must be a valid float or an exception will be thrown. The default value at the start of a SetText call is the size's magnitude which would be 1.0 if no scaling is desired and it is the normal volume.

## Remarks

This command allows to override the normal behavior of the dialogue bleep, pitch and volume done during the [life cycle > Dialogue setup](../../life%20cycle.md#dialogue-setup) phase where the properties will be set using the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) data of the parent's [Entity](../../../Data%20format/Entity.md) and the volume to the size's magnitude. Once this command is processed, every automatic bleep sound done in [life cycle > Letter processing](../../life%20cycle.md#letter-processing) will be using the new values.

It is not possible to change the volume using syntax (1) and (2).
