# CameraFocusTarget
A parameterless method that moves the camera to look at `playerdata[playertargetID]` with some offset to have a zoomed in view of the current player party member being targetted by an enemy action

```cs
private void CameraFocusTarget()
```

## Procedure

- instance.`camoffset` set to (1.0, 2.3, -6.0)
- instance.`camtarget` set to null
- instance.`camtargetpos` set to `playerdata[playertargetID]` position
