# ChargeAttackUnderground
The same than [ChargeAndAttack](ChargeAndAttack.md), but movement is done while underground.

## Frequency meaning
The minimum distance to the player at which no attacks will occur. If the distance falls below this number, a ChargeAndAttack coroutine will be started. NOTE?: if this is above 10.0 or negative, the value is overriden to 2.0.

## DoBehavior
The entity.`digging` is set to true while the `behaviorroutine` isn't in progress. It is set to false otherwise.

The same logic than [ChargeAndAttack](ChargeAndAttack.md) follows, but with the added logic that in order for the ChargeAndAttack to occur, not only the distance check has to be fufilled, but the entity `digtime` must also be above 30.0 or the coroutine won't be called.