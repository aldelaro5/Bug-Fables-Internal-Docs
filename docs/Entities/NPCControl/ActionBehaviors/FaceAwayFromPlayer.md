# FaceAwayFromPlayer
Stops any force move on the entity and causes the entity.`flip` to be set such that the sprite faces the opposite direction of the player horizontally.

## Frequency meaning
None.

## DoBehavior
[StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

If the player is not present, no changes occur on the `flip` value.

The entity.`flip` is set to false when the entity x position is strictly higher than the player. It is set to true otherwise.
