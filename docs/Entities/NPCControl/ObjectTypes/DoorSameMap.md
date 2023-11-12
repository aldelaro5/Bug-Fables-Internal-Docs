# DoorSameMap
A transition zone to an inside of the current map.

## Data Arrays
- `data[0]`: The id of the map.`insides` this transitions to
- `data[1]`: A [music](../../../Enums%20and%20IDs/Musics.md) id to change the current one when entering the inside (the music is changed back to map.`music[0]` when exiting it). This is optional, no change happens if it doesn't exist or it's -1
- `vectordata[0]`: The position to move the player when entering the inside
- `vectordata[1]`: The position to move the player when exiting the inside
- `vectordata[2]`: The `camoffset` to set when entering the inside. This is optional: if the magnitude is below 0.1, no changes happen.
- `vectordata[3]`: The `camangleoffset` to set when entering the inside. This is optional: if the magnitude is below 0.1, no changes happen.
- `vectordata[4]`: OVERRIDEN (stores the original `camoffset` when entering the inside for restoring when exiting it)
- `vectordata[5]`: OVERRIDEN (stores the original `camangleoffset` when entering the inside for restoring when exiting it)
- `vectordata[6]`: The `camlimitpos` to set when entering the inside. This is optional: if the magnitude is below 0.1, no changes happen.
- `vectordata[7]`: The `camlimitneg` to set when entering the inside. This is optional: if the magnitude is below 0.1, no changes happen.

## Additional data
- `regionalflag`: If it's positive, it's the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot that is set to true when the player collides with this object.
- `activationflag`: If it's above 0, the [flag](../../../Flags%20arrays/flags.md) slot that is set to true when the player collides with this object. It is not possible to specify 0 and will be treated the same as not providing a value.

## [CreateEntities](../../EntityControl/CreateEntities.md)
The `insideid` is set to -2 which excludes it from the MapControl.RefreshInsides logic where it could toggle the gameObject enablement if the current `insideid` didn't match (or entity.`hideinside` is true while being in the same `insideid`).

This is also where if `data[1]` exists with a [music](../../../Enums%20and%20IDs/Musics.md) id, the corresponding music is loaded and added to the `musicpreload` list. This list isn't read by anyone, it's just to preload the asset for Unity to load it faster after the map load.

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

## MapControl.RefreshInsides
If we are getting outside while `hideinsides` is true (which is only the case to hide the break room part of the `AntTunnels` [map](../../../Enums%20and%20IDs/Maps.md)), normally, this method would go through all map entities that are inside (or their entity.`hideinside` is true) and disable them and keep the others enabled. 

This object type is excluded from this logic meaning it will never get deactivated from this.

## OnTriggerStay
This does nothing if any of the following is true:
- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true 

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true.

If we aren't in a `minipause`, a MoveInside coroutine is started on the current map with this object as the caller.
