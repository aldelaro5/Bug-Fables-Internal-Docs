# WaterUnderground
The same than [Wander](Wander.md), but the entity immediately starts digging underground before actually wandering.

## Frequency meaning
The same than [Wander](Wander.md).

## SetUp (Only if it's the default behavior)
There is no logic here unlike [Wander](Wander.md) meaning the NPCControl starts with an expired `actioncooldown` and will try to wander on the first admissible DoBehavior cycle.

## SetInitialBehavior (Only if it's the default behavior)
Calls InstantDig on the entity which immediately causes it to dig underground.

## DoBehavior
The entity.`digging` is set to true while the `behaviorroutine` isn't in progress. It is set to false otherwise. This makes sure the NPCControl is `digging` even if behavior switches have happened unless there is `bheaviorroutine` in progress.

After, the same logic than [Wander](Wander.md) follows.
