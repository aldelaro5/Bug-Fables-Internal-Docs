# SetPath
Periodically performs [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) calls on the entity using a fixed configurable interval between moves and the path being made of position nodes defined in `vectordata`.

## Frequency meaning
The time in frames to wait between movement to the next node.

## Additional data
- `vectordata`: The list of position nodes forming the path the AI will attempt to follow.

## SetUp
The entity.`alwaysactive` is set to true.

## DoBehavior
If `returntoheight` is true entity.`height` is set to a lerp from the existing one to entity.`initialheight` with a factor of 0.1.

What happens next depends if all of the following are true:
- The entity isn't in a `forcemove`
- The entity `hitwall` is false
- `vectordata` exists and isn't empty

If all of them are true, then the entity will decide on the next destination to move to. Otherwise, it means the either is in a `forcemove`, its `detect` detects a wall or no nodes are defined in general.

### entity.`forcemove` / entity.`hitwall` / no nodes
- If the entity.`detect` is present, [DetectDirection](../../EntityControl/EntityControl%20Methods.md#detectdirection) is called using the `forcetarget` as the direction.
- If the entity.`hitwall` is true, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called.

### Moving to next destination
If the `actioncooldown` hasn't expired yet, it is decremented by framestep and this cycle ends.

If it has expired, then [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity. `currentmode` is used to determine the next node index to use as the position from `vectordata`. It gets incremented on this cycle when the distance squared between this position and the current node is less than 0.375 (meaning it's essentially at the current one which means it has reached it) and it gets reset to 0 when it reaches the length of `vectordata` so it loops around. The actual call is done with default speed multiplier, a `Walk` walkstate and an `Idle` stopstate.

Finally, `actioncooldown` is set to the frequency and the cycle continues.

## Update (Inactive, every 3 frames)
Normally, when the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove), [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on it, but this behavior is an exception to this where it will not be called here.

## EntityControl.Death
This behavior forbids an [Enemy](../NPCType.md#enemy) with it to drop an item because the behavior uses `vectordata` for its own purposes which interferes with the enemy item drop system.

## [RespawnEnemy](../Notable%20methods/RespawnEnemy.md)
If the behavior is present on the NPCControl, the `actioncooldown` is set to 0.0 and the `currentnode` to 0
