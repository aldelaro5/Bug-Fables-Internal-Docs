# WaterUnderground
The same than [Wander](Wander.md), but the entity digs underground when wandering.

## Frequency meaning
The same than [Wander](Wander.md)

## SetInitialBehavior (Only if it's the default behavior)
Calls InstantDig on the entity which immediately causes it to dig underground.

## DoBehavior
The entity.`digging` is set to true while the `behaviorroutine` isn't in progress. It is set to false otherwise.

After, the same logic than [Wander](Wander.md) follows.
