# TempPlatform
A platform that can only be stood on by the player for a limited amount of time before respawning the player.

## Data Arrays
- `data[0]`: The amount of time in frames that the player can be on the platform
- `data[1]`: ???
- `data[2]`: ???
- `data[3]`: If 0, change the x local position of the entity's `model` to be random between -0.05 and 0.05 which makes it shakes horizontally. If it is 1, the `Shaking` animation clip will play on the entity `anim` when the player `feet` (the ground detector) collides with the platform
- `vectordata[0].x`: The time in frames to wait from the moment a `FlyTrapPlatform` (if it is one) is done with its closing animation and the player is teleported in the air to the moment the fade out occurs.

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

Finally, the entity's `model` tag is set to `Platform`.

## Update
First, the player's `forceclosemove` is set to `hit` TODO: what does this mean ???

What happens after depends on the `hit` value

If it is true:
- If the `actioncooldown` hasn't expired yet, it is decremented by the game's frametime and if `data[3]` is 0, the entity's `model` local position is set to (Random.Range(-0.05, 0.05), 0.0, 0.0) (this makes it shake horizontally)
- If the `actioncooldown` expired on the last update cycle (checked by being above -99999.0), it is set to -100000.0 and if `data[1]` and `data[2]` are both 1, the RespawnPlayer coroutine is started (see in the next section for details).

If it is false:
- `actioncooldown` is set to `data[0]`
- The entity's `model` local position is set to Vector3.zero

### RespawnPlayer
This is a coroutine only used by this object that aims to respawn the player with fade out/fade in.

First, `minipause` is set to true.

If the player's beemerang is present, it is destroyed.

From there, there is a guard clause where most of the logic of this method is performed if this is indeed a `TempPlatform` with the entity's `originalid` being the `FlyTrapPlatform` [animid](../../../Enums%20and%20IDs/AnimIDs.md). It should be noted that under normal gameplay, this caluse is always fufilled because there is no `TempPlatform` in the game that has another animid and this method is only called by the `TempPlatform`'s Update. That being said, no matter if it is fulliled or not, after most of the logic, `minipause` is set back to false and a frame is yieled.

#### RespawnPlayer main logic
As for the main logic, all Hornable objects are gathered and for any of object type `IceCube` that is less than 4.5 away from this object's position + Vector3.up * 10.0, [ShatterDroppletIce](../ShatterDroppletIce.md) is called on their `parent`.

The camera is then forzen by placing the `camtarget` and `camtargetpos` to null followed by the `Bite` sound being played and the `Close` animation clip being played on the entity. This animation clip name corresponds to a [non standard animation](../../EntityControl/Animations/animstate.md#Non-standard-animations). All followers are teleported very close to the player using TeleportFollowers and a yield of 0.07 seconds occurs.

After the yield, all the playerdata entities's `rigid` gets placed in kinematic mode. The player is then placed at this object's position, but very high in the ait by having the y component be 1000.0 before the followers are teleported again using TeleportFollowers. From there, `vectordata[0].x` frames are yielded using a while loop and a temporary timer that decrements by the game's frametime before a frame is yielded until the timer is exhausted.

From there, a fadeout to black is played with a speed of 0.1 and then, a second is yielded.

Once this is over, the player's position is set to its `lastpos` field, the followers are teleported once again using TeleportFollowers and the entity of this object has its `Open` animation clip played. This is again a [non standard animation](../../EntityControl/Animations/animstate.md#Non-standard-animations). The camera is reset to default instantaneously and a frame is yieled.

Finally, a fade in from black is done with a speed of 0.1 and all playerdata's entities's `rigid` are taken out of kinematic mode.

## OnTriggerEnter
If the other collider is the player `feet` (its ground detector) then 2 things occurs:
- If `data[3]` is 1, the `Shaking` animation clip will play on the entity `anim` when the player `feet` (the ground detector) collides with the platform
- `hit` is set to true

## OnTriggerExit
This does nothing if the other gameObject isn't the player or the player entity `feet`.

`hit` is set to false and if we aren't in a `minipause`, the `StayOpen` animation clip is played on the entity `anim`. This is a [non standard animation](../../EntityControl/Animations/animstate.md#non-standard-animations)