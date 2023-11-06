# TriggerSwitch
A trigger zone for a switch ???

## Data Arrays
- `data[0]`: The map entity id this refers to ???. This is optional, anything negative defaults to refer to this ???
- `data[1]`: If it's -1, toggle the `hit` field of the refered entity when entered. If it's 1, it's set to true and false otherwise ???
- `data[2]`: If it's 1, destroy the player `beemerang` when entered ???

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

We resolve the entity `npcdata` this refers to by using `data[0]` as the map entity id, but if it's negative, it defaults to this entity.

The `hit` field of that entity is toggle if `data[1]` is -1 or set to true if `data[1]` is 1 or set to false otherwise.

If the player `beemerang` is present and `data[2]` is 1, the player `beemerang` is destroyed.

## OnTriggerExit
This does nothing if `data[0]` isn't -1 and the other gameObject isn't the player.

The only logic here is `hit` is set to false unless `data[1]` is 0 where it is set to true.