# ChargeAtPlayerFlipSprite
The same then [ChargeAtPlayer](ChargeAtPlayer.md), but right after the [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) call on the entity, the entity.`sprite` has its angles set to (0.0, the y angles facing the player + 90.0, 0.0) with an `overrideonlyflip` which locks the entity.`sprite` facing direction until the charge part of the coroutine is over.

## Frequency meaning
None.

## DoBehavior
The same then [ChargeAtPlayer](ChargeAtPlayer.md), but with the change mentioned above.