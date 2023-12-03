# TriggerSwitch
A trigger zone that controls either its own `hit` value when the player enters or exits the zone or it controls the `hit` value of another NPCControl when the player enters the zone, but not when they exit it.

## Data Arrays
- `data[0]`: If it's -1, the trigger manages its own `hit` value when the player enters or exits the zone. If it's 0 or above, it manages the `hit` value of the map entity whose id is that value when the player enters the zone (not when they exit it). NOTE: any values below -1 are considered invalid because while this trigger `hit` will be set when the player enters the zone, it will not be set when the player exits the zone
- `data[1]`: This has 3 possible valid values (any other values is invalid and causes illogical behaviors):
    - -1: the `hit` value of the refered entity will be toggled when entered. If `data[0]` is -1 on top of this, the `hit` value will always be set to false when the player exits the zone
    - 0: the `hit` value of the refered entity will be set to false when the player enters the zone. If `data[0]` is -1, it will be set to true when the player exits the zone
    - 1: the `hit` value of the refered entity will be set to true when the player enters the zone. If `data[0]` is -1, it will be set to false when the player exits the zone
- `data[2]`: If it's 1, the player.`beemerang` is destroyed when entering the zone

## Additional data
- `regionalflag`: If it's positive, it's the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot that is set to true when the player collides with this object.
- `activationflag`: If it's above 0, the [flag](../../../Flags%20arrays/flags.md) slot that is set to true when the player collides with this object. It is not possible to specify 0 and will be treated the same as not providing a value.

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

- The player isn't present
- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true 

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true when applicable.

We resolve the entity.`npcdata` this refers to by using `data[0]` as the map entity id, but if it's negative, it defaults to this NPCControl.

The `hit` field of that entity is toggled if `data[1]` is -1 or set to true if `data[1]` is 1 or set to false otherwise.

If the player `beemerang` is present and `data[2]` is 1, the player `beemerang` is destroyed.

## OnTriggerExit
This does nothing if `data[0]` isn't -1 and the other gameObject isn't the player.

The only logic here is `hit` is set to false unless `data[1]` is 0 where it is set to true.