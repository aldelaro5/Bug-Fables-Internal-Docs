# FaceAhead
Stops any force move on the entity and causes the entity to face towards the right vector of this entity.

## Frequency meaning
None ???

## SetInitialBehavior (Only if it's the default behavior)
FaceAhead is called on the entity and also invoked it in 0.5 seconds and another time in 1 second.

## DoBehavior
First, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on ethe entity with the facing point being this entity position + its right vector.