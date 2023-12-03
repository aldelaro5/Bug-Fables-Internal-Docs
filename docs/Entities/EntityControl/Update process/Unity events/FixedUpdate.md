# FixedUpdate
This is the FixedUpdate of EntityControl. It only applies when the game isn't paused. Only 2 updates happens here: `forcemove` updates and Leif's position when `leiffly` is true.

## Force move updates
This only applies if `forcemove` is true.

The game first checks the square distance left before its `forcetarget` using `ignorey` as whether or not to consider the y axis. The `forcemove` is considered incomplete unless that square distance is 0.3 or lower.

If it has yet to complete, [Move](../../Notable%20methods/Move.md) is called towards the `forcetarget` with a multiplier of `forcemultiplier` * the same square distance clamped from 0.0 to 1.0 and an [animstate](../../Animations/animstate.md) of `forceanim` during the move. 

After, if `forcejump` is true, the entity will check to see if it should call `Jump`. To do so, `detect` is created via `CreateDetector` if it wasn't already and it is set to look at `forcetarget` with the x and z angles zeroed out and then verify that `hitwall` is true, the entity is `onground` and `jumpcooldown` has already expired. Only if all these conditions are met that Jump is called.

If the whole `forcemove` was instead completed, `forcemove`, `forcejump` and `ignorey` are set to false before calling [StopMoving](../../EntityControl%20Methods.md#StopMoving) with the `forcestop` as the target state.

## Leif fly update
This segment only applies when `leiffly` is true, the player is present and we aren't in an event.

The only thing this segment does is change the local position to a lerp between the current position and the player's position + the player's entity.`spritetransform`.right.normalized * 1.5 + the forward direction of the camera * 0.2 all done with a factor of framestep * 0.05. This basically moves Leif towards the direction he was facing and slightly forward.
