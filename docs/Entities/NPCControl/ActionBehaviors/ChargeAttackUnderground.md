# ChargeAttackUnderground
The same than [ChargeAndAttack](ChargeAndAttack.md), but the behavior always makes the entity `digging` when not attacking and it can only attack after 30.0 frames of `digtime`.

## Frequency meaning
The same than [ChargeAndAttack](ChargeAndAttack.md).

## DoBehavior
The entity.`digging` is set to true (which makes it dig underground) if the `behaviorroutine` isn't in progress (meaning it's not attacking already). It is set to false otherwise.

The same logic than [ChargeAndAttack](ChargeAndAttack.md) follows, but with the added logic that in order for the ChargeAndAttack to occur, not only the distance check has to be fufilled, but the entity `digtime` must also be above 30.0 or the coroutine won't be called.