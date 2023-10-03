# Items

[Items](../Enums%20and%20IDs/Items.md) data are split in 2 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/ItemData`
* `Items` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `ItemData` data

The asset contains one line per [Item](../Enums%20and%20IDs/Items.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|-----:|-------|-------|-------|
|4|Value|int|The buying price of the item in berries (the selling price is half of it)|
|5|Effects|`;` separated list of ItemUse|The list of effects this item has (see below for details)|
|6|Target|AttackArea|The target this item can be used on|

The data will be loaded into `itemdata[0, id, x]` where `id` is the [Item](../Enums%20and%20IDs/Items.md) id and `x` is the loaded index. It should be noted that the id dimension of the array is overprovisioned to exactly 256 elements, this is hardcoded. The first dimension is hardcoded to only have one element, this is also hardcoded.

The attack area must be specified as a string or int representation of an AttackArea enum value. Here are the valid values, anything else is considered invalid:

|Value|Name|
|-----:|----|
|0|SingleEnemy|
|1|AllEnemy|
|2|SingleAlly|
|3|AllParty|
|4|All|
|5|None|
|6|User|

An ItemUse is defined by 2 fields separated by `,`:

|Name|Type|Description|
|----|----|-----------|
|usetype|ItemUsage|The effect the item has|
|value|int|A value that influences the usetype (usually amount or amount of turns)|

The usetype must be specified as a string or int representation of an ItemUsage enum value. Here are the valid values, anything else is considered invalid:

|Value|Name|
|-----:|----|
|0|None|
|1|HPRecover|
|2|TPRecover|
|3|HPRecoverAll|
|4|HPRecoverFull|
|5|TPRecoverFull|
|6|HPUP|
|7|TPUP|
|8|AttackUp|
|9|DefenseUp|
|10|Battle|
|11|Revive|
|12|ReviveAll|
|13|AutoRevive|
|14|CurePoison|
|15|CureFreeze|
|16|CureNumb|
|17|CureSleep|
|18|CureParty|
|19|AddPoison|
|20|AddSleep|
|21|AddNumb|
|22|AddFreeze|
|23|HPto1|
|24|TPto1|
|25|HPto1All|
|26|GradualHP|
|27|GradualTP|
|28|GradualHPParty|
|29|DefUpStat|
|30|AtkUpStat|
|31|Sturdy|
|32|HPUPAll|
|33|MPUP|
|34|ChargeUp|
|35|AtkDownAfter|
|36|CureFire|
|37|CureAll|
|38|TurnNextTurn|
|39|HPorDamage|
|40|CurePoisonAll|

## `Items` data

The asset contains one line per [Item](../Enums%20and%20IDs/Items.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the item|
|1|Unused description|[SetText](../SetText/SetText.md) string|An unused version of the item's description|
|2|Description|[SetText](../SetText/SetText.md) string|The description of the item|
|3|Prepender|[SetText](../SetText/SetText.md) string|The prepender string used with the [Anstring](../SetText/Commands/Individual%20commands/Anstring.md) command for this item|

The data will be loaded into `itemdata[0, id, x]` where `id` is the skill id and `x` is the loaded index.

The unused description is, as its name says, not used in the game. It is safe to leave it empty or a dummy value, but its value must exist for the actual description which needs a valid value.

The prepender is optional: it will not be loaded if it does not exist. which leaves its spot in `itemdata` as null.
