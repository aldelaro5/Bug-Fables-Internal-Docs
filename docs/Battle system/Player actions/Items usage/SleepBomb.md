# `SleepBomb`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|null|`availabletargets[target]`|5 + 2 * amount of `BombPlus` [medals](../../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[currentturn]`|[Pierce](../../Damage%20pipeline/AttackProperty.md)|[Sleep](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
|2|Done for each enemy party member other than the `target` whose position + `freezeoffset` + their `height` in y has a square distance off the object after movement less than 15.5|null|The enemy party member|3 + 2 * amount of `BombPlus` [medals](../../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[currentturn]`|[Pierce](../../Damage%20pipeline/AttackProperty.md)|[Sleep](../../Damage%20pipeline/AttackProperty.md)|empty array|false|

## Item logic

- DeathSmoke particles plays at the `target` enemy party member with a uniform scale of 4.0x
- DoDamage 1 call happens
- For each enemy party member other than the `target` whose position + `freezeoffset` + their `height` in y has a square distance off the object after movement less than 15.5, DoDamage 2 call happens with them
- `Explosion` sound plays
- ShakeScreen is called with an ammount of (0.1, 0.1, 0.1) for 0.15 time
