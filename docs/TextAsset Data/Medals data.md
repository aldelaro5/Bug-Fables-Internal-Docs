# Medals

[Medal](../Enums%20and%20IDs/Medal.md) data are split in 3 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/BadgeData`
* `Ressources/data/BadgeOrder`
* `BadgeName` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `BadgeData` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|2|MP cost|int|The cost of the medal in MP to equip|
|3|Party equipable|bool|Tells if this medal can be equipped to the whole party or not.|
|4|Medal effects|`;` separated list of string[] separated by `,`|The list of effects this medal has (see below for details)|
|5|Value|int|The buying price of the medal in berries|
|7|Crystal Berries value|int|The buying price of the medal in Crystal Berries|

The data will be loaded into `badgedata[id, x]`, where `id` is the [Medal](../Enums%20and%20IDs/Medal.md) id and `x` is the loaded index.

### Medal effects
A medal effect is defined by 2 fields separated by `,`:

|Name|Type|Description|
|----|----|-----------|
|effect|BadgeEffects|The effect the medal has|
|value|int|A value that influences the BadgeEffects (usually an amount)|

The BadgeEffects must be specified as a string or int representation of an BadgeEffects enum value. 

These effects doesn't necessarily tell everything the medal can do. These are a category of effects that can be applied statically without the need for checking if the medal is equipped at a particular moment. The method that applies all of the equipped ones at once is called ApplyBadges.

Here are the valid values, anything else is considered invalid (the apply to column specify if the effect is compatible with member equip or party equip or both):

|Value|Name|Apply to|Value Meaning|Description|
|-----|----|--------|------------|-----------|
|0|None|N/A|None|No effects|
|1|HPUP|Member only|The amount to add to the `hp`|Add the value to the `playerdata`'s `hp`|
|2|TPUP|Party only|The amount to add to the instance.`maxtp`|Add the value to instance.`maxtp`|
|3|HPRecover|N/A|None|UNUSED|
|4|TPRecover|N/A|None|UNUSED|
|5|AttackUp|Member / Party|The amount to add to the attack value(s)|For a member, add the value to the `playerdata`'s `atk` and for the party, add the value to all `playerdata`'s `atk`|
|6|DefenseUp|Member only|The amount to add to the `def`|Add the value to the `playerdata`'s `def`|
|7|LockSkills|Member only|None|Set the `playerdata`'s `lockskills` to true|
|8|Detect|N/A|None|UNUSED (The `Detector` medal has it defined, but no logic backs this effect)|
|9|SpeedUp|Party only|None|Set instance.`speedup` to true (this field is never read making the whole effect practically UNUSED)|
|10|PoisonRes|Member only|The amount to add to the `poisonres`|Add the value to the `playerdata`'s `poisonres`|
|11|SleepRes|Member only|The amount to add to the `sleepres`|Add the value to the `playerdata`'s `sleepres`|
|12|NumbRes|Member only|The amount to add to the `numbres`|Add the value to the `playerdata`'s `numbres`|
|13|FreezeRes|Member only|The amount to add to the `freezeres`|Add the value to the `playerdata`'s `freezeres`|
|14|PoisonAttack|N/A|None|UNUSED (the `PoisonAttacker` medal has it defined, but no logic backs this effect)|
|15|PoisonDefense|N/A|None|UNUSED (the `PoisonDefender` medal has it defined, but no logic backs this effect)|
|16|SleepDefense|N/A|None|UNUSED|
|17|NumbDefense|N/A|None|UNUSED|
|18|FreezeDefense|N/A|None|UNUSED|
|19|AttackMultiply|Member only|The multiplier to multiply the `atk` by|Multiply the `playerdata`'s `atk` by the value. No medals defines this effect under normal gameplay making it practically UNUSED|
|20|DefenseMuliply|N/A|None|UNUSED|
|21|LockItems|Member only|None|Set the `playerdata`'s `lockitems` to true|
|22|LockRelay|Member only|None|Set the `playerdata`'s `locktri` to true|
|23|LockRelayPass|Member only|None|Set the `playerdata`'s `lockrelayreceive` to true|

## `BadgeName` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the medal|
|1|Description|[SetText](../SetText/SetText.md) string|The description of the medal|
|6|Prepender|[SetText](../SetText/SetText.md) string|The prepender string used with the [Anstring](../SetText/Individual%20commands/Anstring.md) command for this medal|

The data will be loaded into `badgedata[id, x]`, where `id` is the [Medal](../Enums%20and%20IDs/Medal.md) id and `x` is the loaded index.

Unlike [Items data](Items%20data.md), the prepender is mandatory.

## `BadgeOrder` data

The asset contains the order to render the medals in the [Medals List Type](../ItemList/List%20Types%20Group%20Details/Medals%20List%20Type.md) where each line contains one field:

|Name|Type|Description|
|----|----|-----------|
|Medal id|int|The [Medal](../Enums%20and%20IDs/Medal.md) id of the medal to render at the position of the line index|

The data will be loaded into `badgeorder[i]` where i is the line index.

It should be noted that this data doesn't just affect the rendering, but also the backing list containing the medals currently in possession. The whole thing is done in RefreshBadgeOrder.
