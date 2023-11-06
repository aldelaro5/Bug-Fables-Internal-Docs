# DoorOtherMap
A loading zone to another map with customisable movement before and after the load.

## Data Arrays
- `data[0]`: The [map](../../../Enums%20and%20IDs/Maps.md) id this leads to
- `vectordata[0]`: The position the party will move to before fading out before the map load
- `vectordata[1]`: The position the party will spawn after the map load
- `vectordata[2]`: The position the party will move to when fading in after the map load

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
This does nothing if any of the following is true:
- The player isn't present
- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true 

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true.

If we aren't in a `minipause`, a TransferMap coroutine is started with `data[0]` as the [map](../../../Enums%20and%20IDs/Maps.md) id, `vectordata[0]` as the position to move to before the load, `vectordata[1]` as the position to spawn after the load, `vectordata[2]` as the position to move to after the load and this object as the caller.
