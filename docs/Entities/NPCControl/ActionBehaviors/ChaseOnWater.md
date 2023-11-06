# ChaseOnWater
The same than [ChasePlayer](ChasePlayer.md), but movement is allowed on water ???

## Frequency meaning
The minimum distance to the player at which no attacks will occur. If the distance falls below this number, a ChargeAndAttack coroutine will be started. NOTE?: if this is above 10.0 or negative, the value is overriden to 2.0.

## DoBehavior
The same than [ChasePlayer](ChasePlayer.md)

## LateUpdate (Non `dummy`, and the entity is `incamera`, but not `iskill`)
If this behavior exists on the NPCControl and the map.`waterfloat` is present, the y component of the position and the entity `starpos` are set to the map.`waterfloat` one clamped from the map.`waterfloat.minwaterfloat` - 0.5 to infinity.