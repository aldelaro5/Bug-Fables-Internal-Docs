# `BurlyBomb`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|null|`availabletargets[target]`|4|null|empty array|false|
|2|Done for each enemy party member other than the `target` whose position + `freezeoffset` + their `height` in y has a square distance off the object after movement less than 15.5|null|The enemy party member|2|null|empty array|false|

## [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) infliction
After DoDamage 1, the `target` enemy party member will have [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on it to inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) for 2 turns with visual effects.

After DoDamage 2, the applicable enemy party member will have the same thing happen, but for 3 turns instead.

## Item logic

- DoDamage 1 call happens and its related [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) call
- `explosion` particles plays at the `target` enemy party member
- For each enemy party member other than the `target` whose position + `freezeoffset` + their `height` in y has a square distance off the object after movement less than 15.5, DoDamage 2 call happens with them and its related [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) call
- `Explosion` sound plays
- ShakeScreen is called with an ammount of (0.1, 0.1, 0.1) for 0.15 time
