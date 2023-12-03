# Follow
Follow is a private method called during [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md) that manages following the `following` entity if it exist, we aren't overriding followers, the entity `overridefollow` isn't active and the entity isn't `dead` or `iskill`.

## Follow logic
There are 3 ways to follow: player and their followers, `tempfollower` and everything not in the first 2 categories. The default follow logic is actually contained in DoFollow described later, but the player and their followers may not call it under some conditions.

### Player and their followers
This branch is applicable if it's not a `tempfollower` and the `following` has the tag `Player` or `PFollower` with the player being present.

The default logic is to set `leiffly` to false, call [Return from action](../EntityControl%20Methods.md#return-from-action) and then call DoFollow. However, there are 3 special cases where this logic isn't performed.

#### Digging
This only applies if the player is digging or started digging. `backsprite` is set to false with a `spin` of 30.0 on the y and the `spritetransform`'s scale is set to be lerped towards 0 with a factor of 0.075. The `transform`'s position is changed directly towards `following` + the forward direction of the camera / 2.0. This is done because it bypasses the physics engine. The `rigid` is actually set to kinematic without gravity and the `ccol` is disabled so the physics engine no longer can control the follower when digging. The `shadow` is disabled if it wasn't and finally, [StopMoving](../EntityControl%20Methods.md#stopmoving) is called with the default `basestate` (-1) which completely stops any velocity.

#### Flying
This only applies if the player is flying. First, `backsprite` is set to false, same with `overrridejump` and `overrideanim` to get back control on the flying animations. [StopForceMove](../EntityControl%20Methods.md#stopforcemove) is called with no [animstate](../Animations/animstate.md) and no smoothing which halts any current coroutine force move if one existed.

From there, if the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) was `Beetle`, his [animstate](../Animations/animstate.md) is changed to 102, his position changed directly to be the player's + the forward direction of the camera * 0.2 and his `flip` changed to the player's. This forces Kabbu to face the same direction and to be positioned where the player is. On the other hand, if it's not `Beetle`, then it is assumed to be `Moth` and as such, his `rigid` gets its gravity disabled, `overrideanim` becomes true, his [animstate](../Animations/animstate.md) becomes 101, his `flip` becomes the player's and `leiffly` becomes true. This is needed during [FixedUpdate](../Update%20process/Unity%20events/FixedUpdate.md) to update his position smoothly towards the player.

#### Shielding
During bubble shield, the follow logic is managed differently by ShieldMove (tempf to false since this is a player follower). It forces the entity to face towards the player, [StopForceMove](../EntityControl%20Methods.md#stopforcemove) is called with `walkstate` and no smoothing before the position is set directly via a lerp to the player's + an offset with a factor of framestep * 0.25. This offset is calculated by using the [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) number as an offset in 0.5 units in x and 0.2 units in z which makes the entity go very close to the player, but with enough spacing to prevent z fighting.

#### Default
If none of the above applies, then the default logic mentioned earlier applies which ends with a DoFollow call.

### Temp followers
There are 3 starting logic branches for a `tempfollower`:

* The player is shielding: then ShieldMove with tempf to true is called. The only change with the player follower version is that the offset is now the player's `spritetransform` right direction * 0.75 + the camera foward direction * 0.5.
* The player is flying: the [animstate](../Animations/animstate.md) is set to `basestate`, the `rigid` gravity is disabled, `flip` is set to to the last player in playerdata's `flip` (basically the last party member in the follow chain) and the position is set directly to be the last party member's position + (+-0.35, 0.75, 0.1) * `tempfollowerid` + 1 which will offset the position to be up forward and left/right. The x sign is determined by the player's `flip`.
* Everything else: DoFollow is called

After this is done, the `rigid` gravity is enabled, `nodigpart` is set to true and `digging` is set to whether the player is digging or has started digging. If we are indeed `digging`, the position is set directly to the player's + the forward direction of the camera * 0.1.

## Default logic
If none of the other 2 branches applies, DoFollow is called as the only execution for this follow.

## DoFollow
DoFollow is a private method that houses the default logic of a typical follow that entities will use most of the time. It applies unless the game's `overridefollower` is enabled, `overridefollow` is true, the entity is `dead` or `iskill`.

From there, unless there's a `springcooldown` and `jumpcooldown` is above 15.0, the `rigid` velocity is set to to the current one in x/z and in y to the clamp of the current one from -20.0 to `jumpheight`. After, [FaceTowards](../EntityControl%20Methods.md#facetowards) is called with the `following` position. The rest of the method only happens every 2 frames so if we're in an odd frame, this is done.

If we aren't however, this is where the actual follow management is handled. 

### Movement logic
We first gather the horizontal square distance between the `transform` and the `following` which will be used to decide what to do for this follow. 

### Too far handling
The first thing that is checked is if the entity is too far from `following` which it is if the square distance in x/z is higher than `followlimit` (20.0 by default) OR we aren't in a minipause and the y actual distance is higher than the map's followerylimit. In that case, the position is changed directly to the `following`'s + the forward direction of the camera * 0.1 and the velocity is zeroed out.

### Decide whether to move
From there, we can now check the if entity should not move or move towards `following`. The entity will attempt to move towards it only if the square distance is higher than `followdistance` (by default 2.0). However, this check changes to half of the `followdistance` if CloseMove returns true. This method only returns true under the following conditions:

* The player is present and not dashing
* One or more of the following is true:
    * We are currently in a stealth section ([flags](../../../Flags%20arrays/flags.md) 401), the map's closemove is true and this entity is part of the `mainparty`
    * This entity is part of the `mainparty` and the player has a parent
    * The player has forceclosemove to true

The result is that the entity will attempt to be twice as close than normal under these conditions.

#### Follow movement
If we decided to move towards following, MoveTowards is called which starts a `forcemove` to `following` + the forward direction of the camera * `followoffset`. This is however halved if CloseMove returned true earlier. The `forcemove` is done with a multiplier of 1.25 unless CloseMove returned true which makes it (the square distance - 1) / 2.0. The rest are left default with `ignore_y` to true.

From there we go into determining if we should Jump, but only if `detect` is present. If it is, [DetectDirection](../EntityControl%20Methods.md#detectdirection) is called which aligns the wall detector. From there, there are 2 potential scenario that will make the entity jump (only one of them needs to be true):

* The entity is `onground` and a raycast hit occurs from `transform`.position + `detect`.transform.forward.normalized + Vector3.up * 0.3 directed down with a max distance of 2.5 only hitting layers Ground and NoDigGround. This raycast basically ensures that there is a wall that can be jumped over by the entity, but it doesn't make use of `hitwall` to check this and rather by raycasting manually using the `detect` direction.
* The entity is `onground`, it has `hitwall` and the actual distance between the `transform` and `following` is higher than 0.5 and the horizontal square distance between the 2 is higher than `followjump` (0.3 normally). This basically means both detector reports ground and wall while the horizontal and vertical distance are high enough to warrant jumping.

Jump is not called if neither applies or `detect` isn't present.

#### Follow stop movement
If we decided to not move towards `following`, [StopForceMove](../EntityControl%20Methods.md#stopforcemove) is called with the `basestate` with smoothing. After, the `deltavelocity` is zeroed out if the [animstate](../Animations/animstate.md) isn't `Walk`.
