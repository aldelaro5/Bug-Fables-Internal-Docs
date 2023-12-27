# SetDefaultCamera
This method resets the default camera's properties and loads the offset / position from the battle fields:

```cs
public static void SetDefaultCamera(bool reset)
```

## Parameters

- `reset`: Tells if the battle fields should be reset to their default value too

## Procedure
The method does the following assignement to the camera fields:

- `camspeed` to 0.1
- `camoffset` to battle.`camoffset`
- `camangleoffset` to `battlecamangle`
- `camtarget` to null
- `camtargetpos` to battle.`campos`

It is also possible to send true as a parameter to do a more aggressive version which also resets the battle fields:

- `camspeed` to 0.1
- `camoffset` to `battlecampos`
- `camangleoffset` to `battlecamangle`
- `camtarget` to null
- `camtargetpos` to Vector3.zero
- battle.`camoffset` to `battlecampos`
- battle.`campos` to Vector3.zero