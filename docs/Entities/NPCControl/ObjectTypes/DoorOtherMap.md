# DoorOtherMap
A loading zone to another map with customisable movement before and after the load.

## Data Arrays
- `data[0]`: The [map](../../../Enums%20and%20IDs/Maps.md) id this leads to
- `data[1]`: If 1, the `camoffset` is set to `vectordata[3]` when fading in after the map load. This is optional, if it doesn't exist, no changes to the camera is done
- `data[2]`: If 1, the `camangleoffset` is set to `vectordata[4]` when fading in after the map load. This is optional only if `data[1]` doesn't exist too
- `data[3]`: If 1, the map.`camlimitpos` and map.`camlimitneg` are set to `vectordata[5]` and `vectordata[6]` respectively when fading in after the map load. This is optional only if `data[1]` doesn't exist too
- `data[4]`: If 1, there will be no movement before the map load and instead, all `playerdata` entity.`rigid` will have their gravity disabled in kinematic mode. This is optional, if it doesn't exist, movement happens as normal
- `vectordata[0]`: The position the party will move to before fading out before the map load if `data[4]` isn't 1
- `vectordata[1]`: The position the party will spawn after the map load
- `vectordata[2]`: The position the party will move to when fading in after the map load
- `vectordata[3]`: The `camoffset` is set when fading in after the map load if `data[1]` is 1. This is optional only if `data[1]` doesn't exist
- `vectordata[4]`: The `camangleoffset` is set when fading in after the map load if `data[2]` is 1. This is optional only if `data[1]` doesn't exist
- `vectordata[5]`: The `camlimitpos` is set when fading in after the map load if `data[3]` is 1. This is optional only if `data[1]` doesn't exist
- `vectordata[6]`: The `camlimitneg` is set when fading in after the map load if `data[3]` is 1. This is optional only if `data[1]` doesn't exist
- `emoticonoffset.x`: If it's higher than 0.1, the movement after the map load is done with a JumpTo call on the player with this value as the height. If it's lower, a standard [MoveTowards](../../EntityControl/EntityControl%20Methods.md#MoveTowards) is used on the player entity instead

## Additional data
- `regionalflag`: If it's positive, it's the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot that is set to true when the player collides with this object.
- `activationflag`: If it's above 0, the [flag](../../../Flags%20arrays/flags.md) slot that is set to true when the player collides with this object. It is not possible to specify 0 and will be treated the same as not providing a value.

## [CreateEntities](../../EntityControl/CreateEntities.md)
The `insideid` is set to -2 which excludes it from the MapControl.RefreshInsides logic where it could toggle the gameObject enablement if the current `insideid` didn't match (or entity.`hideinside` is true while being in the same `insideid`).

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

## MapControl.LateUpdate
After 20 frames has passed since the load of the map and every 2 frames, this object type has some special logic where the gameObject and its entity.`emoticon`'s gameObject gets disabled if the player gets 15.0 (ignoring y) or more away from the object and enabled if it gets back within 15.0.

## OnTriggerStay
This does nothing if any of the following is true:

- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true (never happens under normal gameplay)

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true.

If we aren't in a `minipause`, a TransferMap coroutine is started with `data[0]` as the [map](../../../Enums%20and%20IDs/Maps.md) id, `vectordata[0]` as the position to move to before the load, `vectordata[1]` as the position to spawn after the load, `vectordata[2]` as the position to move to after the load and this object as the caller.
