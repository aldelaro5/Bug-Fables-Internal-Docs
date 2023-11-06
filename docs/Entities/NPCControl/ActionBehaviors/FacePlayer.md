# FacePlayer
Stops any force move on the entity and causes the entity to face towards the player if it is present.

## Frequency meaning
None ???

## SetInitialBehavior (Only if it's the default behavior)
FacePlayer is called on the entity and also invoked it in 0.5 seconds and another time in 1 second.

## DoBehavior
First, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove).

Then, if the player is present, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity with the facing point being the player position.

## Interact
If this behavior is the second one in the `behaviors` array (the `inrange` one), [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity with the facing point being the player position.