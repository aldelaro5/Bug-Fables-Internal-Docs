# SetDefaultCamera
This method resets the default camera's properties and loads the offset / position from the battle fields:

```cs
public static void SetDefaultCamera(bool reset)
```

## Parameters

- `reset`: Tells if the battle fields should be reset to their default value too

## Procedure
The method does the following assignement to the camera fields:

|Name|Value|
|---:|-----|
|`camspeed`|0.1|
|`camoffset`|battle.`camoffset`|
|`camangleoffset`|`battlecamangle`|
|`camtarget`|null|
|`camtargetpos`|battle.`campos`|

It is also possible to send true as a parameter to do a more aggressive version which also resets the battle fields:

|Name|Value|
|---:|-----|
|`camspeed`|0.1|
|`camoffset`|`battlecampos`|
|`camangleoffset`|`battlecamangle`|
|`camtarget`|null|
|`camtargetpos`|Vector3.zero|
|battle.`camoffset`|`battlecampos`|
|battle.`campos`|Vector3.zero|
