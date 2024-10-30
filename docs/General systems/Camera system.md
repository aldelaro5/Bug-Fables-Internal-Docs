# Camera system
This game's camera is fully managed by a state machine that can be interfaced via fields in MainManager. It allows to easilly position and angle the camera in many intuitives ways such as either to place it somewhere or to look at a specific object and follow it.

Most of the camera system is managed by MainManager via some fields that act as a state machine and via the RefreshCamera method which is invoked on FixedUpdate (after LoadEverything is done). The fields allows RefreshCamera to determine how the camera should be positioned and they can be changed by the game at will.

However, the current MapControl is what has the biggest influence on the camera initially. It's possible to further configure the camera at runtime after the map is loaded (and it's even possible to override anything the map does), but the intent is mostly to have MapControl direct the initial behavior while the game can decide to override it if needed. For more information on what MapControl does to the camera system, check the [map camera systems](../MapControl/Camera%20system.md) documentation.

## Camera structure
The only scene the game uses is where the Cameras are defined in the game. It has the following GameObject structure (this is rooted from the scene):

```
-> MainCam
----> Main Camera
-------> GUICamera
----------> RenderTexture (initially disabled)
-------> 3DGUI
```

The camera system is mostly concerned with the "MainCam" object since it is a rooted object so moving or rotating it will cause changes to all the other cameras. In other words, moving or rotating this object is all that's needed to position and angle the camera. "Main Camera" is used to relatively displace the cameras by setting its local position.

Most of the time, "MainCam" is addressed by accessing the parent of "Main Camera".

### Main Camera
The "Main Camera" object is quite possibly the most important GameObject in the game because it not only has the main Camera of the game attached, but it also is where the only MainManager is. Its camera is initialised when the game boots into MainManager.`MainCamera`

This is the camera that is used to render everything except anything in the following layers:

- `UI`
- `3DUI`

This separation is done because UI objects have different rendering criteria than regular objects in the game. For the most part though, this is the camera where most of the game can be seen and it is a regular perspective mode camera (meaning it can render depths in the scene).

This is also the camera that renders the skybox as its set as the clear flags (the other 2 cameras are set to depths only).

### GUICamera
This camera only renders elements on the `UI` layer and it is special because unlike the `MainCamera`, this one is in orthographic mode. It means that depth is effectively absent from the vies of this camera: everything is rendered as if it was in 2D. This is perfect for most of the UI in the game since they don't have depth to it, they just need to be on screen with relatively fixed sizes and positioning.

It can be accessed from the MainManager.`GUICamera` field and is used to render most UI elements in the game including [SetText](../SetText/SetText.md) lines.

#### RenderTexture
The `GUICamera` has a `RenderTexture` GameObject under it that has a CRT shader. This is how the game can have Termacade games look the way they do in conjunction with another featrue: downsampling.

There is a game setting for downsampling, but it isn't exposed. It's only used by a method on MainManager called SetRenderTexture which is mostly used as part of Termacada games.

```cs
public static void SetRenderTexture(int downsampleindex)
```

Essentially, `RenderTexture` remains disabled until SetRenderTexture is called with a non zero value where it will get enabled and have its material's mainTexture set to a new 1080p RenderTexture. During this process, the `MainCamera` gets its rect set to a smaller area depending on how much downsampling is desired (Termacade games asks for 20% downsampling).

To turn this off, SetRenderTexture needs to be called with 0 as downsampleindex which will cause `RenderTexture` to be disabled and the camera to revert to normal. The reason this is childed to the `GUICamera` is because that rendering is effectively a 2D image rendered on the camera so it shouldn't have depths rendered on it. It's effectively a way to apply post processing effectively and render it as if it was how the game was being rendered.

### 3DGUI
This camera only renders elements on the `3DUI` layer. Unlike the other 2 cameras, it's not possible to access it directly, but there's not a need to do this. This is because this camera doesn't need to be rendered directly onto like the `GUICamera` and it isn't needed to address it for it to function.

However, it remains important because it is what allows UI elements to be rendered with depths. This camera unlike the `GUICamera` is configured in perspective mode so any UI elements that needs to be rendered with depths to them uses this camera. This is mainly needed to render UI that needs to look relatively close to other objects in games such as icons and emoticons.

[SetText](../SetText/SetText.md) lines can render on it using the [triui](../SetText/Individual%20commands/Triui.md) command.

## Camera fields
The camera's logic is driven by a state machine in MainManager that uses the following fields to determine how to position it and angle it:

|Name|Type|Description|
|---:|----|----------|
|camtarget|Transform|If not null, the camera will attempt to look at the transform and follow them when they move|
|camspeed|float|The lerp factor to use when moving the camera. 1.0 means the camera moves instantly to its position. The value should always be higher than 0.0 because it would otherwise make the camera stuck in place. The closer this value is to 1.0, the less delayed the camera movement will be through time|
|camtargetpos|Vector3?|If not null, but `camtarget` is null, the camera will attempt to move to this position directly|
|screenshake|Vector3|If it's not Vector3.zero, it repsents a random range of displacement to add to the camera's local position once its base local position has been determined. The range of displacement for each components is between 0.0 - `X` and `X` where `X` is the value of this vector|
|camoffset|Vector3|The main part of the local position to set on the camera. The local position of the camera will be set to a lerp from its current local position to this field + `camoffset2` with a factor of `camspeed`. Essentially, it allows a relative displacement from its current position|
|camoffset2|Vector3|The secondary part of the local position to set on the camera. This value + `camoffset` will be the destination of the local postion lerp, but as a separate field. It should only be used to have an offset that acts on top of `camoffset`|
|camanglechange|bool|If true and instance.`map`.`rotatecam` is false or we are in a map's [inside](../MapControl/Insides.md), the LerpAngle factor used when angling the camera is going to be `camanglespeed` instead of `camspeed`|
|camanglespeed|float|If camanglechange is true and instance.`map`.`rotatecam` is false or we are in a map's [inside](../MapControl/Insides.md), this will be the LerpAngle factor used when angling the camera instead of `camspeed`|
|camangleoffset|Vector3|If instance.`map`.`rotatecam` is false or we are in a map's [inside](../MapControl/Insides.md), this represents the destination angles of the LerpAngle used to angle the camera|

These fields however aren't the only ones that determines the camera angles and position. Part of it is handled by the [MapControl specific portion](../MapControl/Camera%20system.md). which contains fields to further configure this system.

## RefreshCamera
This method is responsible for using all the fields mentioned above and the MapControl fields to position and angle the camera. It is only called by FixedUpdate continously after LoadEverything is done. Here is what the method does.

### "MainCam" positioning
The first thing RefreshCamera does is set the position of "MainCam".

There are 3 cases (checked in order, the first one that applies will be done):

- `camtarget` isn't null: "MainCam" will be positioned towards this transform position
- `camtarget` is null, but `camtargetpos` isn't: "MainCam" will be positioned towards this position
- Both `camtarget` and `camtargetpos` are null: "MainCam" position remains unchanged

This is the logic that happens for the first 2 cases:

- If `map` is null, it means the map is currently being loaded so it can't influence the camera. The "MainCam" position is set to a lerp from the existing one to the destination with a factor of `camspeed`
- Otherwise (`map` isn't null), it depends on `map`.`tetherdistance` and `insideid`:
    - If `map`.`tetherdistance` is above 0.0 (it has a value defined) and we aren't in an [inside](../MapControl/Insides.md), the "MainCam" position is set to a lerp from the existing one with a factor of `camspeed`. The destination of this lerp is the destination clamped from `map`.`camlimitneg` to `map`.`camlimitpos` limited by a radius of `map`.`tetherdistance` from `map`.`actualcenter` as the center point
    - Otherwise, the "MainCam" position is set to a lerp from the existing one to the destination clamped from `map`.`camlimitneg` to `map`.`camlimitpos` with a factor of `camspeed`

### `MainCamera` local postion adjustements
After, the "MainCamera" local position gets to a lerp from its existing one to `camoffset` + `camoffset2` with a factor of `camspeed`. However, if `screenshake` isn't Vector3.zero, a random vector is generated between 0.0 - `screenshake` and `screenshake` and it is added to the `MainCamera` local position (after the lerp is done).

### Angling "MainCam" and `MainCamera`
Finally, the last step in RefreshCamera is to angle the camera.

There's 2 ways the camera can be angled. Either by looking at the `map`.`actualcenter` or by setting the angles to `camangleoffset`. The former is only performed if the following conditions are fufilled:

- `map` isn't null (a map isn't being loaded)
- `map`.`rotatecam` is true (the map is configured to rotate towards a center point)
- `insideid` is -1 (we aren't in an [inside](../MapControl/Insides.md))

If the following are fufilled this is how the camera is angled:

- `MainCamera` is set to LookAt `map`.`actualcenter`
- "MainCam" is set to LookAt `map`.`actualcenter`
- If `map`.`tieYtoplayer` is true, "MainCam" x angle is set to 0.0

Otherwise, this is how the camera is angled:

- The factor to use with LerpAngle is determined to be `camanglespeed` if `camanglechange` is true, `camspeed` if it's false
- Each components of `MainCamera` angles are set to a LerpAngle from the current one to 0.0 with the factor determined above
- Each components of "MainCam" angles are set to a LerpAngle from the current one to `camangleoffset` with the factor determined above
