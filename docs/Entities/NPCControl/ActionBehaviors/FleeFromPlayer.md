# FleeFromPlayer
Stops any force move on the entity and have it move towards the opposide direction from the player.

## Frequency meaning
None.

## DoBehavior
If the entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, if we aren't `tattling` or we are in `pause` (NOTE: this is likely a bug as it allows movement when paused), [Move](../../EntityControl/Notable%20methods/Move.md) is called on the entity at the direction opposite of the player position with normal speed multiplier and with the `Idle` [animstate](../../EntityControl/Animations/animstate.md).