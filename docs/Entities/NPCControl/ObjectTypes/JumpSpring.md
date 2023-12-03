# JumpSpring
A spring that makes an entity jump up high via [Jump](../../EntityControl/EntityControl%20Methods.md#jump) when touched or makes the entity jump straight to a specific point via [JumpTo](../../EntityControl/EntityControl%20Methods.md#JumpTo) with the option to trigger an inside transition via a [DoorSameMap](DoorSameMap.md) present on the map without physically moving the player.

## Data Arrays
- `data[0]`: If 1, the spring will make an entity jump to `vectordata[1]` via [JumpTo](../../EntityControl/EntityControl%20Methods.md#JumpTo) when an entity touches the spring. Otherwise, the spring will make the entity jump up high when touched via [Jump](../../EntityControl/EntityControl%20Methods.md#jump) 
- `data[1]`: If 1, jumps are only performed when the entity touching the spring isn't `onground` and its `rigid` y velocity is negative (meaning, the entity must be falling onto the spring from the air above). If 0, this check is disabled, any other value is considered invalid and will prevent any jumps to be performed when touching the spring
- `data[2]`: If not negative and `data[0]` is 1, this is a map entity id corresponding to a [DoorSameMap](DoorSameMap.md) that will be triggered when the spring jump occurs
- `vectordata[0].x`: The height of the jump. If `data[0]` is 0, this must be above 1.0 or it will be overriden to the default `jumpheight` of the entity that collides with this object
- `vectordata[1]`: The position to jump to when touched if `data[0]` is 1
- `vectordata[2].x`: If `data[0]` is 1, the jump multiplier clamped from 1.0 to 99.0. This is optional, if it doesn't exist, the default is 1.0.

## Additional data
- `boxcol`: Required to be present with valid data

## Setup
A few adjustements occurs:
- The entity.`activeonpause` is set to true
- `internalvector` is initialised to 2 elements with the first being `boxcol`.center and the second being (0.0, 555.0, 999.0) which is an offscreen position.
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity and with all constraints frozen
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
The `boxcol`.center is set to the first `internalvector` (the original `boxcol`.center) if the instance's `itempicked` is false or to the second one (the offscreen position) if it's true. This just places the `boxcol`.center at an hardcoded offscreen position if we are in the process of a [CheckItem](../Notable%20methods/CheckItem.md).

## OnTriggerStay
This does nothing if any of the following are true:
- We are in a `pause`
- The current [map](../../../Enums%20and%20IDs/Maps.md) is `RubberPrisonGym` while the other gameObject y position is not higher than this object y position
- The other gameObject is the player and it is not free (ignoring fly)

The entity.`overrideflip` is set to true and the other gameObject's entity is obtained.

For an actual jump to be registered on the spring, the following must all be true:
- The other entity exists 
- The other entity is not `iskill`
- `data[1]` is 0 or `data[1]` is 1 and the other entity isn't `onground` and its `rigid` has a negative y velocity
- The other entity `npcdata` doesn't exist or it does, but it's not an `Object` or the entity is an [item entity](../../EntityControl/Item%20entity.md)

Otherwise, if the other gameObject tag is `BeetleHorn`, `BeetleDash` or `Icecle` the logic is limited to redoing a call to `BounceAnim` (stopping entity.`bounceanim` if it was in progress, but reassigning it to a new call). The call happens on the entity with a push amount of 1.2, a time of 50.0 frames and a squash speed of 0.2 all done gradually. This visually renders a bouncy spring, but it won't move the entity.

### Spring jump logic
If the other gameObject is the player and the distance between it and this object is less than 10.0, the `Boing0` sound is played at 0.75 volume.

From there, the logic differs depending on `data[0]`. At the end of either branch, `BounceAnim` is called on the entity (stopping entity.`bounceanim` if it was in progress, but reassigning it to a new call). The call is done with a push amount of 1.2, a time of 50.0 frames and a squash speed of 0.2 all done gradually. It only makes the spring visually jump.

#### `data[0]` is 1 ([JumpTo](../../EntityControl/EntityControl%20Methods.md#JumpTo) logic)
If the other gameObject is the player, nothing happens here if the `collisionammount` is 100 or above (meaning we already entered this since the last LateUpdate). If it's less than 100 however:
- The player position is set to this object position + (0.0, 0.75, 0.0)
- [JumpTo](../../EntityControl/EntityControl%20Methods.md#JumpTo) is called on the player with the position being `vectordata[1]`, the height being `vectordata[0].x` and the multiplier being 1.0 unless `vectordata[2].x` exists which will make it be the multiplier instead clamped from 1.0 to 99.0.
- If `data[2]` isn't negative, MoveInside will be called on the map with the NPCControl that resolves to the map entity id of `data[2]` as the caller without move (this implies that NPCControl is a [DoorSameMap](DoorSameMap.md))
- `collisionammount` is set to 110.0 (which prevents to reenter this branch until the end of the next LateUpdate)

#### `data[0]` isn't 1 ([Jump](../../EntityControl/EntityControl%20Methods.md#Jump) logic)
- The other entity.`springcooldown` is set to true
- The other entity.`jumpcooldown` is set to 30.0
- If the other gameObject is the player, CancelAction is called on the player keeping the beemerang. On top of this, if `itempicked` is true (we are in a [CheckItem](../Notable%20methods/CheckItem.md) process), the player entity.`onground` is set to true
- Unless we set.`onground` to true earlier, the following occurs:
  - The entity.`springcooldown` is set to true
  - [Jump](../../EntityControl/EntityControl%20Methods.md#Jump) is called on the other entity with the height being its `jumpheight` unless `vectordata[0].x` is above 1.0 and the other doesn't have an `npcdata` that is a [RollingRock](RollingRock.md) where `vectordata[0].x` is used as the height instead

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.