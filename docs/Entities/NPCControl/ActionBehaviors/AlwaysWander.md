# AlwaysWander
Same as [wander](Wander.md), but with a few differences ???

## Frequency meaning
The same as [wander](Wander.md) ???

## SetUp (Only if it's the default behavior)
There is no logic here unlike [wander](Wander.md).

## DoBehavior
The same than [wander](Wander.md), but with 2 differences:
- Before going into the main logic when it applies, if we are on `pause` or the player is `tattling`: [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called and this DoBehavior cycle is ended early.
- When deciding if we should stop in the entity.`forcemove` branch, there is an early exit where if we are on `pause` or the player is `tattling`: [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called and this DoBehavior cycle is ended early.

TODO: This isn't clear on how this affects the behavior vs wander
