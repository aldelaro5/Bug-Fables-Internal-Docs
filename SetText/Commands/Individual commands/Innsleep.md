# Innsleep

Play the inn sleeping cutscene with configurable final positions and camera controls then yield until it is over.

## Syntax

(1)

````
|innsleep|
````

(2)

````
|innsleep,xposition,yposition,zposition|
````

(3)

````
|innsleep,xposition,yposition,zposition,true|
````

(4)

````
|innsleep,xposition,yposition,zposition,true,xcamlimitneg,ycamlimitneg,zcamlimitneg,xcamlimitpos,ycamlimitpos,zcamlimitpos|
````

## Parameters

### `xposition`: float

The x position to place the party after the cutscene. This must be a valid float or an exception will be thrown.

### `yposition`: float

The y position to place the party after the cutscene. This must be a valid float or an exception will be thrown.

### `zposition`: float

The z position to place the party after the cutscene. This must be a valid float or an exception will be thrown.

### `true`

Send this value to restore the music after the cutscene to the main map's music. Any other value means to not do it and keep the silence, but it must still be specified to respect the position of the parameters in syntax (4).

### `xcamlimitneg`: float

The z component of the lower camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

### `ycamlimitneg`: float

The y component of the lower camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

### `zcamlimitneg`: float

The z component of the lower camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

### `xcamlimitpos`: float

The x component of the upper camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

### `ycamlimitpos`: float

The y component of the upper camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

### `zcamlimitpos`: float

The z component of the upper camera position limit to set after the cutscene. This must be a valid float or an exception will be thrown. This is mostly useful if the new position of the party is located outside of the current camera limits.

## Remarks

This command plays the whole cutscene including playing the "Inn" SFX, starting the iris wipe transition and even includes the healing. It can be seen as a very specialized version of an [Events](../../../Enums%20and%20IDs/Events.md), but the cutscene itself isn't one as it has its own coroutine called InnSleep.

Before playing the cutscene, the current textbox will have its shrink animation start if it exist and it has a DialogueAnim component.

The coroutine call is tracked via the chaptername field which is normally used to play chapter intros, but it is reused here (this is safe because its value will be null by the time the coroutine is over). The game will yield until the coroutine is over and once it is, [End](End.md) is set to true.
