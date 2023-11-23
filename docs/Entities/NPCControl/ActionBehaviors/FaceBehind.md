# FaceBehind
Stops any force move on the entity and causes the entity to face towards the left vector of this entity (the opposite direction of their x axis direction)

## Frequency meaning
None.

## SetInitialBehavior (Only if it's the default behavior)
FaceBehind is called on the entity and it is also invoked in 0.5 seconds and another time in 1 second.

## DoBehavior
[StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity with the facing point being this entity position - its right vector.