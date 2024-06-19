# Default toss action
This action is a toss item action and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

This is an action logic defined that normally doesn't happen under normal gameplay. It would happen for any toss item (has a [Battle](../../Battle%20flow/Action%20coroutines/UseItem.md#battle) itemUse), but whose [item](../../../Enums%20and%20IDs/Items.md) id does not belong in the list of action backed items documented here. In that case, the logic defaults to the following [DoDamage](../../Damage%20pipeline/DoDamage.md) call:

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done for each player party member whose `hp` is above 0 and who doesn't have an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)|null|The player party member|10|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
|2|Done for each enemy party member who doesn't have a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition and whose [enemy id](../../../Enums%20and%20IDs/Enemies.md) isn't `Ahoneynation` or `Abomihoney`|null|The enemy party member|10|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
