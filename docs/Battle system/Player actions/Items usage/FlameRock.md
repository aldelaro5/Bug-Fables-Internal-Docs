# `FlameRock`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Item logic
The folowing [DoDamage](../../Damage%20pipeline/DoDamage.md) call happens:

|attacker|target|damageammount|property|overrides|block|
|---|---|---|---|---|---|
|null|`availabletargets[target]`|2, but if the target has a [Magic](../../Damage%20pipeline/AttackProperty.md) in its [weakness](../../Actors%20states/Enemy%20features.md#weakness), it's 4 instead|[Fire](../../Damage%20pipeline/AttackProperty.md)|empry array|false|
