# `Ice`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Item logic
The folowing [DoDamage](../../Damage%20pipeline/DoDamage.md) call happens:

|attacker|target|damageammount|property|overrides|block|
|---|---|---|---|---|---|
|null|`availabletargets[target]`|2|[Freeze](../../Damage%20pipeline/AttackProperty.md)|empry array|false|
