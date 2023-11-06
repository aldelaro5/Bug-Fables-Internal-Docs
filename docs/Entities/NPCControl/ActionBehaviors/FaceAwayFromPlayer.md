# FaceAwayFromPlayer
Stops any force move on the entity and causes the entity `flip` to be set such that it faces the opposite direction than the player horizontally.

## Frequency meaning
None ???

## DoBehavior
First, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove).

Then, if the player is present, the entity's `flip` is set to true if this entity x position is strictly higher than the player's. It is set to false otherwise.
