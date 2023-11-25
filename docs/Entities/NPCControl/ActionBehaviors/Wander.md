# Wander
Periodically performs [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) call after a random amount of time passed on the entity to a random position or the closest possible before unable to move further towards it. The range of movement, maximum time period to wait between wanders and the max distance to move to between wanders are all configurable.

## Frequency meaning
The maximum time in frames to wait before moving again. The minimum is always the 1/3 of the maximum.

## Additional data
- `radiuslimit`: The max distance in which the entity is allowed to wander from its current position
- `wanderradius`: The x/z radius from entity.`startpos` in which the entity can wander to
- `teleportradius`: The max distance the entity can be away from its `startpos` before being teleported to its `startpos`

## SetUp (Only if it's the default behavior)
The `actioncooldown` is set to 120 frames.

## DoBehavior
The entity.`rigid` is unlocked with [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid) if it was in kinematic or it didn't had gravity.

Then, If `returntoheight` is true, entity.`height` is set to a lerp from the existing one to entity.`initialheight` with a factor of 0.1

From there, there is an early exit path if the entity is in a `forcemove` and any of the following is true:
- entity.`hitwall` is true
- entity.`onground` is false
- entity.`detect` is present and [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) from the entity.`forcetarget` returns false

When the above is fufilled, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called with the `basestate` and this DoBehavior cycle is ended early. This is because the game concluded it is no longer possible to move further towards this position.

If we are cleared to continue, there are 4 possible branches this DoBehavior can takes (only the first one that applies is taken checked in this order):
- `actioncooldown` expired (the main wandering logic)
- `maxtries` is exactly 10
- The distance between entity.`startpos` and this position is higher than the `teleportradius` or `maxtries` is 20 or above (meaning it went too far away or there's too much failed wander attempts)
- The entity is in a `forcemove`

If none of the branches above are taken, it means to wait before the next wander:
- `walkcooldown` is set to 0.0
- `trycount` is set to 0
- `actioncooldown` is decremented by the game's frametime
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity

### `actioncooldown` expired (main wandering logic)
This branch is essentially the main logic that manages the movement to the next point. It works by attempting to generate a random vector that satifies the `wanderradius` and `radiuslimit` constraints. The former dictates the range of where the entity can move and the latter dictates the maximum distance the next point can be.

The attempted next vector is 1/3 of a random one between (-`wanderradius`, 0.5, -`wanderradius`) and (`wanderradius`, 0.5, `wanderradius`) which is then added to the entity.`startpos` as the `wanderradius` is relative to it. When generated, the entity.`moverotater` is set to look at it regardless if it's accepted or not.

If the vector is less than `radiuslimit` away from the current position (meaning it's not too far to move to), then it is accepted which makes the following happen:
- [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move to the vector at default everything with the exception that the y component is ignored
- The entity.`detect` is set to LookAt the vector
- `actioncooldown` is set to be random between 1/3 of the frequency and the frequency itself
- `maxtries` is set to 0

If the vector is denied, this is where `trycount` comes into play as it is incremented here. 

The purpose of this field and `maxtries` is to regulate the timeout logic. If it reaches 50 or above, `actioncooldown` is set to be random between 1/3 of the frequency and the frequency itself once again, `trycount` is reset to 0 and `maxtries` is incremented. It's `maxtries` that can start to be more aggressive if it gets too high (this is outlined in 2 other branches).

### `maxtries` is exactly 10
[MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move to its `startpos` ignoring the y component with a 0.3 multiplier using the default entity.`walkstate` and entity.`basestate`. Finally, `maxtries` is set to 11. This times out the main logic such that if too much failed wander attempts happened, the entity moves back to its `startpos`.

### Too far or too much failed wander attempts
This is basically a warp to force the entity to teleport to its `startpos`:
- DeathSmoke particles are played at this position if the entity is `incamera`
- The position is set to the entity.`startpos`
- The entity.`rigid` velocity is zeroed out
- DeathSmoke particles are played at this position if the entity is `incamera` again
- `maxtries` is set to 0
- `actioncooldown` is set to 100.0

### The entity is in a `forcemove`
[StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity if `walkcooldown` is higher than 60.0 and the entity.`rigid` x and z components have a magnitude less than half its `speed`. 

Whether or not this happens, `walkcooldown` is incremented by the game's frametime. This basically forces the entity to at least wander for 60 frames and to stop whenever the entity slows down its movement enough.