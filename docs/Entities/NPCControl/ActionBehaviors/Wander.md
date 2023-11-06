# Wander
Periodically performs [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) call after a random amount of time passed on the entity to a random position. The range of movement, maximum time period to wait between move and the max distance to move to between them are all configurable.

## Frequency meaning
The maximum time in frames to wait before moving again. The minimum is always the 1/3 of the maximum.

## SetUp (Only if it's the default behavior)
The `actioncooldown` is set to 120 frames.

## DoBehavior
First, the entity `rigid` is unlocked with [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid) if it was in kinematic or it didn't had gravity.

Then, If the NPCControl `returntoheight` is true, `height` is set to a lerp from the existing one to `initialheight` with a factor of 0.1

From there, there is an early exit path if the entity is in a `forcemove` and `hitwall` is true or `onground` is false or the `detect` is present and [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) from the `forcetarget` returns false. When this happens, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove)  is called with the `basestate` and this DoBehavior cycle is ended early.

From there, there are 4 branches this DoBehavior takes (only the first one that applies is taken checked in this order):
- `actioncooldown` expired: see the section below about the main logic
- `maxtries` is exactly 10: [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move to its `startpos` ignoring the y component with a 0.3 multiplier using the default `walkstate` and `basestate`. Finally, `maxtries` is set to 11. This times out the main logic such that if too much failed wander attempts happened, the entity goes back to its `startpos`
- The distance between the entity `startpos` and this position is higher than the `teleportradius` or `maxtries` is 20 or above (meaning it went too far away or there's too much failed wander attempts): the following occurs (this is basically a warp to force the entity to move to its `startpos`):
  - DeathSmoke particles are played at this position if the entity is `incamera`
  - The position is set to the entity `startpos`
  - The entity `rigid` velocity is zeroed out
  - DeathSmoke particles are played at this position if the entity is `incamera` again
  - `maxtries` is set to 0
  - `actioncooldown` is set to 100.0
- The entity is in a `forcemove`: [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity if `walkcooldown` is higher than 60.0 and the entity `rigid` x and z components have a magnitude less than half its `speed`. Whether or not this happens, `walkcooldown` is incremented by the game's frametime. This basically forces the entity to at least wander for 60 frames and to stop whenever the entity slows down its movement enough.
- If none of the branches above are taken, `walkcooldown` is set to 0.0, `trycount` to 0, `actioncooldown` is decremented by the game's frametime and [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity

### Main wandering logic
This branch is essentially the main logic that manages the movement to the next point. It works by attempting to generate a random vector that satifies the `wanderradius` and `radiuslimit` constraints. The former dictates the range of where the entity can move and the latter dictates the maximum distance the next point can be.

The attempted next vector is a 1/3 of a random one between (-`wanderradius`, 0.5, -`wanderradius`) and (`wanderradius`, 0.5, `wanderradius`) which is then added to the entity `startpos` as the `wanderradius` is relative to it. When generated, the `moverotater` is set to look at it regardless if it's accepted or not.

If the vector is less than `radiuslimit` away from the current position (meaning it's not too far to move to), then it is accepted which makes the following happen:
- [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move to the vector at default everything with the exception that the y component is ignored
- The entity `detect` is set to LookAt the vector
- `actioncooldown` is set to be random between 1/3 of the `frequency` and the `frequency` itself
- `maxtries` is set to 0

If the vector is denied, this is where `trycount` comes into play as it is incremented here. 

The purpose of this field and `maxtries` is to regulate the timeout logic. If it reaches 50 or above, `actioncooldown` is set to be random between 1/3 of the `frequency` and the `frequency` itself once again, `trycount` is reset to 0 and `maxtries` is incremented. It's `maxtries` that can start to be more aggressive it reaches too high (this is outlined in the steps list above).