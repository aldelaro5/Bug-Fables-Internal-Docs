# Medals

[Medal](../Enums%20and%20IDs/Medal.md) data are split in 3 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/BadgeData`
* `Ressources/data/BadgeOrder`
* `BadgeName` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `BadgeData` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|2|MP cost|int|The cost of the medal in MP to equip|
|3|Party equipable|bool|Tells if this medal can be equipped to the whole party or not.|
|4|Item effects|`;` separated list of ItemUse|The list of item effects this medal has (see below for details)|
|5|Value|int|The buying price of the medal in berries|
|7|Crystal Berries value|int|The buying price of the medal in Crystal Berries|

The data will be loaded into `badgedata[id, x]`, where `id` is the [Medal](../Enums%20and%20IDs/Medal.md) id and `x` is the loaded index.

An ItemUse is defined by 2 fields separated by `,`:

|Name|Type|Description|
|----|----|-----------|
|usetype|ItemUsage|The effect the item has|
|value|int|A value that influences the usetype (usually amount or amount of turns)|

The usetype must be specified as a string or int representation of an ItemUsage enum value. Here are the valid values, anything else is considered invalid:

|Value|Name|
|-----|----|
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

## `BadgeName` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the medal|
|1|Description|[SetText](../SetText/SetText.md) string|The description of the medal|
|6|Prepender|[SetText](../SetText/SetText.md) string|The prepender string used with the [Anstring](../SetText/Commands/Individual%20commands/Anstring.md) command for this medal|

The data will be loaded into `badgedata[id, x]`, where `id` is the [Medal](../Enums%20and%20IDs/Medal.md) id and `x` is the loaded index.

Unlike [Items data](Items%20data.md), the prepender is mandatory.

## `BadgeOrder` data

The asset contains the order to render the medals in the [Medals List Type](../ItemList/List%20Types%20Group%20Details/Medals%20List%20Type.md) where each line contains one field:

|Name|Type|Description|
|----|----|-----------|
|Medal id|int|The [Medal](../Enums%20and%20IDs/Medal.md) id of the medal to render at the position of the line index|

The data will be loaded into `badgeorder[i]` where i is the line index.

It should be noted that this data doesn't just affect the rendering, but also the backing list containing the medals currently in possession. The whole thing is done in RefreshBadgeOrder.
