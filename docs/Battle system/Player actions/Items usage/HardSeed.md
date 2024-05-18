# `HardSeed`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Item logic
The folowing [DoDamage](../../Damage%20pipeline/DoDamage.md) call happens:

|attacker|target|damageammount|property|overrides|block|
|---|---|---|---|---|---|
|null|`availabletargets[target]`|2|null|empry array|false|
