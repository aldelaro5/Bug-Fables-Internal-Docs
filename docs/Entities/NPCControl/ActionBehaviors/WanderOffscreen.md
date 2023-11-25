# WanderOffscreen
The same than [Wander](Wander.md), but movements are allowed when inactive or paused.

## Frequency meaning
The same than [Wander](Wander.md).

## Additional data
The same than [Wander](Wander.md).

## SetUp (Only if it's the default behavior)
There is no logic here unlike [Wander](Wander.md).

## DoBehavior
The same than [Wander](Wander.md).

## LateUpdate (Not a `dummy` and entity is `incamera`)
If this behavior exists (as default or `inrange`), `startlife` is less than 300.0 frames and the entity.`activeonpause` is false, the entity.`activeonpause` and entity.`alwaysactive` are set to true. This allows to have the [Wander](Wander.md) be in effect even during inactive updates.