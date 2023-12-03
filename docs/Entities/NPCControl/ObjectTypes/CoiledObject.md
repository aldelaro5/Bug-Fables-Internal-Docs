# CoiledObject
A coil that can trap another map entity within using a configurable local position and requires to be hit by a player ability to untrap the object. It can optionally set a [flag](../../../Flags%20arrays/flags.md) slot to true when hit.

## Data Arrays
- `data[0]`: The map entity id of the object being trapped
- `data[1]`: A [flag](../../../Flags%20arrays/flags.md) slot that is set to true whenever the object gets hit by a player ability. This is optional, it is not specified if the value is negative
- `vectordata[0]`: The local position of the trapped object

## Additional data
- `boxcol`: Required to be present with valid data
- `regaionalflag`: If not negative, the corresponding [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot is set to true when the coil gets hit

## Setup
A few adjustements occurs:

- entity.`alwaysactive` is set to true
- gameObject's isStatic is set to true
- entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
Updates are disabled as long as `hit` is false.

There's 2 cases. Either the `boxcol` is enabled or disabled.

If it is enabled, then the entity.`model` local scale is set to a lerp from the existing one to (100.0, 100.0, 120.0) with a factor of the 1/5 of the game's frametime. This only increases the z scaling because the base scale of the `CoilyVine` prefab is 100.0 uniform, but the pivot point is also rotated such that the z axis is vertical so the overall effect is this grows it vertically. If the z component of the entity.`model` local scale reached higher than 115.0, the `boxcol` gets disabled which ends this update cycle.

If the `boxcol` was disabled (happens when the vine grew enough vertically), then the entity.`model` local scale is set to a lerp from the existing one to (100.0, 100.0, 0.0) with a factor of the 1/5 of the game's frametime. This makes it scales down vertically to become effectively flat upwards.

## LateUpdate (NaN in position component)
This is logic that applies only to the `trapped` `Object` reffered by this `CoiledObject` and not the `CoiledObject` itself.

If the NPCControl is `trapped`, there is a special case where the game will find the first `Object` NPCControl that is also a `CoiledObject` with a `data[0]` matching the `mapid` (this is checked by entity id order). When found, the local position of the `Object` is set to the `vectordata[0]` of the map entity found and its entity.`rigid` is locked via [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#lockrigid). Nothing happens if none of them are found.

NOTE: it is unknown why the game seems to specifically care about a NaN positive with a `trapped` object. This doesn't seem to happen under normal gameplay.

## LateUpdate (Not a `dummy` and the entity is `incamera`)
- If `moveobj` isn't initialised yet, `hit` is false and the entity isn't `iskill`, then `moveobj` is initialised to the NPCControl at `data[0]` as `mapid` with some adjustements:
    - Its `trapped` is set to true
    - Its entity.`rigid` is locked via [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#lockrigid)
    - It gets childed to this entity.`sprite`
    - Its scale is set to Vector3.one
    - Its local position is set to `vectordata[0]`

## OnTriggerEnter
Nothing happens if all of the following are true:

- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash` and `Icecle` while it's not the player `beemerang`
- `moveobj` is not present (meaning this coil doesn't contain its object)
- `hit` is true

The `Coiled` sound is played on this entity and HitPart particles are played at this position + (0.0, 0.5, 0.0).

The following occurs on the entity of the `moveobj` (the trapped object):

- The `rigid` is unlocked via [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid), but its velocity gets zeroed out
- The `npcdata.trapped` is set to false
- The `onground` is set to false
- It gets childed to the current map
- The local scale is set to Vector3.one

If `data[1]` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to true.

If the `regaionalflag` isn't negative, the corresponding [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot is set to true.

If the other collider was the player `beemerang`, its `hit` is set to true followed by the `WoodHit` sound being played.

Finally, `hit` is set to true.

## Effects of being `trapped`
This field is specific to this object and the object being `trapped` receives some specific effects:

- If it's a [PushRock](PushRock.md), OnTriggerEnter becomes disabled preventing the rock to be moved by Kabbu's Horn Slash
- [CheckItem](Item.md#checkitem) prevents any `trapped` [Item](Item.md) from being obtained by the player.
- It won't receive active NPCControl updates unless entity.`activeonpause` is true (and even if it is, it can only receive updates if it's an [Enemy](../NPCType.md#enemy) with one of the 3 active update override cases)
- Any inactive updates will skip setting the position to entity.`startpos`
- The entity's [UpdateVelocity](../../EntityControl/Update%20process/UpdateVelocity.md) effects are only active every 2 frames when the npcdata.`entitytype` isn't [NPC](../NPC.md)
- For all active EntityControl update on the entity, the first child of the `spritetransform`'s enablement is not updated
