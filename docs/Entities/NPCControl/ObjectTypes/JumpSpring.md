# JumpSpring
A spring that makes the player jump up hight when touched.

## Data Arrays
- `data[0]`: If 1, ???
- `data[1]`: If 0, ??? Otherwise, if 1 ???
- `data[2]`: If not negative, this is a map entity id that will be moved inside ???
- `vectordata[0].x`: The height of the jump.
- `vectordata[1]`: The position to jump to when touched ???
- `vectordata[2].x`: The jump multiplier clamped from 1.0 to 99.0 ??? This is optional, if it doesn't exist, the default is 1.0.

## Setup
A few adjustements occurs:
- The entity's `activeonpause` is set to true
- `internalvector` is initialised to 2 elements with the first being `boxcol.center` and the second being (0.0, 555.0, 999.0)
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity and with all constraints frozen
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
The `boxcol`'s center is set to the first `internalvector` if the instance's `itempicked` is false or to the second one if it's true TODO: what ???

## OnTriggerStay
This does nothing if any of the following are true:
- We are in a `pause`
- The current [map](../../../Enums%20and%20IDs/Maps.md) is `RubberPrisonGym` while the other gameObject position is not higher than this object
- The other gameObject is the player and it is not free (ignoring fly)

First, the entity `overrideflip` is set to true and the other gameObject's entity is obtained.

There are 2 possible branches that follows and they are mutually exclusive. The first one is the main logic and it needs the following to be all true:
- The other entity exists and it's `iskill`
- `data[0]` is 0 or `data[1]` is 1 and the other entity isn't `onground` and its `rigid` has a negative y velocity
- The other entity `npcdata` doesn't exist or it does, but it's not an `Object` or it is and its entity is an `item`

Otherwise, the second one is if the other gameObject tag is `BeetleHorn`, `BeetleDash` or `Icecle` and on that one, the logic is limited to redoing a call to `BounceAnim`. The details of that logic appears at the end of the main one.

### Main logic
First, if the other gameObject is the player and the distance between it and this object is less than 10.0, the `Boing0` sound is played at 0.75 volume.

Then, if `data[0]` is 1 and the other gameObject is the player, there is only logic if the `collisionammount` is less than 100:
- The player position is set to this object position + (0.0, 0.75, 0.0)
- JumpTo is called on the player with the position being `vectordata[1]`, the height being `vectordata[0].x` and the multiplier being 1.0 unless `vectordata[2].x` exists which will make it be the multiplier instead clamped from 1.0 to 99.0.
- If `data[2]` isn't negative, MoveInside without move will be called on the map on the NPCControl that resolves to the map entity id of `data[2]`
- `collisionammount` is set to 110.0

Otherwise:
- The other entity `springcooldown` is set to true
- The other entity `jumpcooldown` is set to 30.0
- If the other gameObject is the player, CancelAction is called on the player keeping the beemerang. On top of this, if `itempicked` is true, the player entity `onground` is set to true
- Unless we set `onground` to true earlier, the following occurs:
  - The entity `springcooldown` is set to true
  - [Jump](../../EntityControl/EntityControl%20Methods.md#Jump) is called on the other entity with the height being its `jumpheight` unless `vectordata[0].x` is above 1.0 and the other doesn't have an `npcdata` that is a [RollingRock](RollingRock.md) where `vectordata[0].x` is used as the height instead

Finally, the entity `bounceanim` is stopped if it was in progress followed by assigning it to a new call of `BounceAnim` on the entity with a push amount of 1.2, a time of 50.0 frames and a squash speed of 0.2 all done gradually.
