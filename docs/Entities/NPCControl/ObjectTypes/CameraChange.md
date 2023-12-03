# CameraChange
A trigger zone to change the camera's properties.

## Data Arrays
- `data[0]`: If 1, the `camoffset` is changed by this object when triggered
- `data[1]`: If 1, the map camera limits are changed by this object when triggered
- `data[2]`: If 1, the `camspeed` is changed by this object when triggered
- `data[3]`: If 1, the `camangleoffset` is changed by this object when triggered
- `data[4]`: If 1, the `camtarget` is changed by this object when triggered
- `data[5]`: The [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) whose entity's transform is the value the `camtarget` will be set to when `data[4]` is 1
- `data[6]`: If 1, the `camtargetpos` is changed by this object when triggered
- `data[7]`: If 1, the `camanglespeed` is changed by this object when triggered. This is optional
- `vectordata[0]`: The `camoffset` to set if `data[0]` is 1
- `vectordata[1]`: The map `camlimitpos` (upper limit bound) to set if `data[1]` is true. NOTE: if the magnitude isn't above 0.1, `camlimitpos` and `camlimitneg` are reset to default when this is triggered and `data[1]` is true
- `vectordata[2]`: The map `camlimitneg` (lower limit bound) to set if `data[1]` is true. NOTE: if the magnitude isn't above 0.1, `camlimitpos` and `camlimitneg` are reset to default when this is triggered and `data[1]` is true
- `vectordata[3].x`: The `camspeed` to set if `data[2]` is 1 which will also set `changecamspeed` to true
- `vectordata[3].y`: The `camanglespeed` to set if `data[7]` is 1 which will also set `camanglechange` to true
- `vectordata[4]`: The `camangleoffset` to set to if `data[3]` is 1
- `vectordata[5]`: The `camtargetpos` to set to if `data[6]` is 1

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
This does nothing if any of the following is true:
- The other gameObject isn't the player
- The player isn't free (ignoring flying) and its entity is not `digging`

If the above conditions are fufilled, the camera data is changed accoriding to the data logic described above.
