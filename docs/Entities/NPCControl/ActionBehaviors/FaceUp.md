# FaceUp
Stops any force move on the entity and causes the entity to face towards the forward vector of this entity (usually away from the camera).

## Frequency meaning
None.

## SetInitialBehavior (Only if it's the default behavior)
FaceUp is called on the entity and it is also invoked in 0.5 seconds and another time in 1 second.

## DoBehavior
[StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity with the facing point being this entity position + its forward vector.