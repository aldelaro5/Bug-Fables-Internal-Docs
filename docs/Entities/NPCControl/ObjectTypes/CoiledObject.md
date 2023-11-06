# CoiledObject
An object inside a coiled vine ???

## Data Arrays
- `data[0]`: The entity map id of the object trapped
- `data[1]`: A [flag](../../../Flags%20arrays/flags.md) slot that is set to true whenever the object gets hit by a player ability. This is optional, it is not specified if the value is negative
- `vectordata[0]`: the local position of the trapped object ???

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
Updates are disabled as long as `hit` is false.

There's 2 cases. Either the `boxcol` is enabled or disabled.

If it is enabled, then the entity's `model` local scale is set to a lerp from the existing one to (100.0, 100.0, 120.0) with a factor of the 1/5 of the game's frametime. If the z component of the `model`'s local scale reached higher than 115.0, the `boxcol` gets disabled which ends this update cycle.

If the `boxcol` was disabled, then the entity's `model` local scale is set to a lerp from the existing one to (100.0, 100.0, 0.0) with a factor of the 1/5 of the game's frametime.

## LateUpdate (NaN in position component)
This is logic that applies only to the `trapped` `Object` reffered by this `CoiledObject` and not the `CoiledObject` itself.

If the NPCControl is `trapped`, there is a special case where the game will find the first `Object` NPCControl that is also a `CoiledObject` with a `data[0]` matching the `mapid` (this is checked by entity id order). When found, the local position of the `Object` is set to the `vectordata[0]` of the map entity found and its entity.`rigid` is locked via [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#lockrigid). Nothing happens if none of them are found.

TODO: interesting, why would we care about having NaN on CoiledObject ???

## LateUpdate (Not a `dummy` and the entity is `incamera`)
- If `moveobj` isn't initialised yet, `hit` is false and the entity isn't `iskill`, then `moveobj` is initialised to the map entity at `data[0]` with some adjustements:
  - Its `trapped` is set to true
  - Its `rigid` is locked via [LockRigid(true)](../EntityControl/EntityControl%20Methods.md#lockrigid)
  - It gets childed to this entity `sprite`
  - Its scale is set to Vector3.one
  - Its local position is set to `vectordata[0]`

## OnTriggerEnter
Nothing happens if all of the following are true:
- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash` and `Icecle` while it's not the player `beemerang`
- `moveobj` is not present TODO: where does it come from ???
- `hit` is true

First, the `Coiled` sound is played on this entity and HitPart particles are played at this position + (0.0, 0.5, 0.0).

Then, the following occurs on the entity of the `moveobj`:
- The `rigid` is unlocked via [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid), but its velocity gets zeroed out
- The `npcdata.trapped` is set to false
- The `onground` is set to false
- It gets childed to the current map
- The local scale is set to Vector3.one

If `data[1]` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to true.

If the `regaionalflag` isn't negative, the corresponding [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot is set to true.

If the other collider was the player `beemerang`, its `hit` is set to true followed by the `WoodHit` sound being played.

Finally, `hit` is set to true.
