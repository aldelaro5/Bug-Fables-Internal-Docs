# FleeFromPlayer
Stops any force move on the entity and have it move towards the opposide direction from the player.

## Frequency meaning
None ???

## DoBehavior
First, if the entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, if we aren't `tattling` or we are in `pause` (NOTE: this is likely a bug), [Move](../../EntityControl/Notable%20methods/Move.md) is called on the entity at the relative direction opposite of the player with normal speed with the `Idle` [animstate](../../EntityControl/Animations/animstate.md).