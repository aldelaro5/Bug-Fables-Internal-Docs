# WanderOnWater
The same than [WanderNoWarp](WanderNoWarp.md), but the y position is adjusted according to the map.`waterfloat`.

## Frequency meaning
The same than [WanderNoWarp](WanderNoWarp.md) ???

## DoBehavior
The same than [WanderNoWarp](WanderNoWarp.md).

## LateUpdate (Non `dummy`, and the entity is `incamera`, but not `iskill`)
If this behavior exists on the NPCControl and the map.`waterfloat` is present, the y component of the position and the entity `starpos` are set to the map.`waterfloat` one clamped from the map.`waterfloat.minwaterfloat` - 0.5 to infinity.