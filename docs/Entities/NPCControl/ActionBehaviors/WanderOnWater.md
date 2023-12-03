# WanderOnWater
The same than [WanderNoWarp](WanderNoWarp.md), but the y position is adjusted according to the map.`waterfloat` if one exists.

## Frequency meaning
The same than [Wander](Wander.md).

## Additional data
The same than [Wander](Wander.md).

## SetUp (Only if it's the default behavior)
There is no logic here like [WanderNoWarp](WanderNoWarp.md), but unlike [Wander](Wander.md).

## DoBehavior
The same than [WanderNoWarp](WanderNoWarp.md).

## LateUpdate (Non `dummy`, and the entity is `incamera`, but not `iskill`)
If this behavior exists on the NPCControl (either default or `inrange`) and the map.`waterfloat` is present, the y component of the position and the entity.`starpos` are set to the map.`waterfloat` one clamped from the map.`waterfloat.minwaterfloat` - 0.5 to infinity. This forces the NPCControl to be positioned on the same level than the map.`waterfloat` object which is the object that defines where the water's surface is located.