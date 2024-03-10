# Items

[Items](../Enums%20and%20IDs/Items.md) data are split in 2 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/ItemData`
* `Items` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `itemdata`

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

### DoItemEffect
The ItemUse effects are applied in a static method on MainManager called DoItemEffect which applies no matter how the items was used (out or in battle). There are extra logic in battle managed by [UseItem](../Battle%20system/Battle%20flow/Action%20coroutines/UseItem.md).

```cs
public static int DoItemEffect(ItemUsage type, int value, int? characterid)
```
The type and value comes from the ItemUse and the characterid is the `playerdata` index that is using the item. While the characterid is nullable, in practice, null is never sent and sending it leads to an exception being thrown the vast majority of the time.

Once the effect is applied, if `battle` is not null (a battle is in progress), UpdateConditionIcons is called on it which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true). Finally, the method returns the final value (which may be changed by the effects).

Here's all the ItemUsage's effects from this method:

#### `HPRecover` or `Revive`

- The `Heal` sound is played
- If we are instance.`inbattle`, value is incremented by the amount of `HealPlus` [medal](../Enums%20and%20IDs/Medal.md) equipped on `lastitemuser`
- The `hp` of the characterid player party member is increased by value then clamped from 0 to its `maxhp`
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `HPorDamage`

- If `battle` is null (no battle is in progress):
    - value is incremented by the amount of `HealPlus` [medal](../Enums%20and%20IDs/Medal.md) equipped on `lastitemuser`
    - An RNG check is performed with 2 possible outcomes at 59% and 41% respectively:
        - 59%: The `Heal` sound is played followed by the `hp` of the characterid player party member being increased by value then clamped from 0 to its `maxhp`
        - 41%: The `Damage0` sound is played followed by the `hp` of the characterid player party member being decreased by value then clamped from 1 to its `maxhp`
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `HPRecoverAll`

- The `Heal` sound is played
- If we are instance.`inbattle`, value is incremented by the amount of `HealPlus` [medal](../Enums%20and%20IDs/Medal.md) equipped on `lastitemuser`
- For each player party member that isn't `eatenby` with an `hp` above 0:
    - The `hp` of the player party member is increased by value then clamped from 0 to its `maxhp`
    - If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the player party member, an `AddPoison` effect is processed with the corresponding characterid being the player party member's. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `ReviveAll`
The same then `HPRecoverAll`, but the player party members don't need an `hp` above 0 for the effects to apply.

#### `TPRecover`
- The `Heal2` sound is played
- instance.`tp` is increased by value then clamped from 0 to instance.`maxtp`
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `TurnNextTurn`

- The `Heal3` sound is played
- If we are instance.`inbattle`, the `moreturnnextturn` of the characterid player party member is incremented

#### `HPUP`

- The `StatUp` sound is played
- An `HP` stat bonus with the amount being the value is added to the characterid player party member (learn more [here](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses))
- [ApplyStatBonus](../Battle%20system/ApplyStatBonus.md) is called

#### `TPUP`

- The `StatUp` sound is played
- An `TP` stat bonus with the amount being the value is added to the party (learn more [here](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses))
- [ApplyStatBonus](../Battle%20system/ApplyStatBonus.md) is called

#### `HPUPAll`

- The `StatUp` sound is played
- An `HP` stat bonus with the amount being the value is added to the party (learn more [here](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses))
- [ApplyStatBonus](../Battle%20system/ApplyStatBonus.md) is called

#### `AttackUp`

- The `StatUp` sound is played
- An `Attack` stat bonus with the amount being the value is added to the characterid player party member (learn more [here](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses))
- [ApplyStatBonus](../Battle%20system/ApplyStatBonus.md) is called
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `AtkDownAfter`
The `atkdownonloseatkup` of the characterid player party member is set to true

#### `ChargeUp`

- The `charge` of the characterid player party member is incremented then clamped from 0 to 3
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `DefenseUp`

- The `StatUp` sound is played
- An `Defense` stat bonus with the amount being the value is added to the characterid player party member (learn more [here](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses))
- [ApplyStatBonus](../Battle%20system/ApplyStatBonus.md) is called
- If we are instance.`inbattle` and the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3

#### `AddPoison`
This effect only applies if we are instance.`inbattle` and the characterid player party member doesn't have the [Sturdy](../Battle%20system/Actors%20states/BattleCondition/Sturdy.md) condition

- The `Poison` sound is played
- If the characterid player party member doesn't have any `PoisonResistance` or `ResistAll` [medals](../Enums%20and%20IDs/Medal.md) equipped:
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called with the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition on the characterid player party member for value amount of turns. The amount is overriden to 9999999 if the characterid player party member has an `EternalPoison` [medals](../Enums%20and%20IDs/Medal.md) equipped and if not, it is increased by 1 if it has the `WeakStomach` medal equipped

#### `AddNumb`
This effect only applies if we are instance.`inbattle` and the characterid player party member doesn't have the [Sturdy](../Battle%20system/Actors%20states/BattleCondition/Sturdy.md) condition

- The `Shock` sound is played
- If the characterid player party member doesn't have any `NumbResis` or `ResistAll` [medals](../Enums%20and%20IDs/Medal.md) equipped:
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called with the [Numb](../Battle%20system/Actors%20states/BattleCondition/Numb.md) condition on the characterid player party member for value amount of turns

#### `AddFreeze`
This effect only applies if we are instance.`inbattle` and the characterid player party member doesn't have the [Sturdy](../Battle%20system/Actors%20states/BattleCondition/Sturdy.md) condition

- The `Freeze` sound is played
- If the characterid player party member doesn't have any `FreezeResistance` or `ResistAll` [medals](../Enums%20and%20IDs/Medal.md) equipped:
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called with the [Freeze](../Battle%20system/Actors%20states/BattleCondition/Freeze.md) condition on the characterid player party member for value amount of turns

#### `AddSleep`
This effect only applies if we are instance.`inbattle` and the characterid player party member doesn't have the [Sturdy](../Battle%20system/Actors%20states/BattleCondition/Sturdy.md) condition

- The `Sleep` sound is played
- If the characterid player party member doesn't have any `SleepyResistance` or `ResistAll` [medals](../Enums%20and%20IDs/Medal.md) equipped:
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called with the [Sleep](../Battle%20system/Actors%20states/BattleCondition/Sleep.md) condition on the characterid player party member for value amount of turns

#### `CureFire`

- The `Heal3` sound is played
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the characterid player party member to remove the [Fire](../Battle%20system/Actors%20states/BattleCondition/Fire.md) condition
- If we are instance.`inbattle` and the characterid player party member's `firepart` isn't null, it is destroyed

#### `CureAll`

- The `Heal3` sound is played
- A bunch of [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) calls happens in sucession on the characterid player party member to remove the following [condition](../Battle%20system/Actors%20states/Conditions.md):
    - [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md)
    - [Sleep](../Battle%20system/Actors%20states/BattleCondition/Sleep.md)
    - [Freeze](../Battle%20system/Actors%20states/BattleCondition/Freeze.md)
    - [Numb](../Battle%20system/Actors%20states/BattleCondition/Numb.md)
    - [Fire](../Battle%20system/Actors%20states/BattleCondition/Fire.md)
    - [Inked](../Battle%20system/Actors%20states/BattleCondition/Inked.md)
    - [Sticky](../Battle%20system/Actors%20states/BattleCondition/Sticky.md)
- If we are instance.`inbattle`, the following happens on the characterid player party member:
    - If `firepart` isn't null, it is destroyed
    - [BreakIce](../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the battleentity
    - `isasleep` is set to false
    - `isnumb` is set to false

#### `CurePoisonAll`

- The `Heal3` sound is played
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on each of the player party members to remove the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition

#### `CurePoison`

- The `Heal3` sound is played
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the characterid player party member to remove the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition

#### `CureSleep`

- The `Heal3` sound is played
- The characterid player party member's `isasleep` is set to false
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the characterid player party member to remove the [Sleep](../Battle%20system/Actors%20states/BattleCondition/Sleep.md)

#### `CureFreeze`

- The `Heal3` sound is played
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the characterid player party member to remove the [Freeze](../Battle%20system/Actors%20states/BattleCondition/Freeze.md)
- If we are instance.`inbattle`, [BreakIce](../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) is called on the battleentity of the characterid player party member

#### `CureNumb`

- The `Heal3` sound is played
- The characterid player party member's `isnumb` is set to false
- [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the characterid player party member to remove the [Numb](../Battle%20system/Actors%20states/BattleCondition/Numb.md)

#### `CureParty`

- The `Heal3` sound is played
- A bunch of [RemoveCondition](../Battle%20system/Actors%20states/Conditions%20methods/RemoveCondition.md) calls happens in sucession on each of the player party members that isn't `eatenby` with an `hp` above 0 to remove the following [condition](../Battle%20system/Actors%20states/Conditions.md):
    - [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md)
    - [Sleep](../Battle%20system/Actors%20states/BattleCondition/Sleep.md)
    - [Freeze](../Battle%20system/Actors%20states/BattleCondition/Freeze.md)
    - [Numb](../Battle%20system/Actors%20states/BattleCondition/Numb.md)
    - [Fire](../Battle%20system/Actors%20states/BattleCondition/Fire.md)
    - [Inked](../Battle%20system/Actors%20states/BattleCondition/Inked.md)
    - [Sticky](../Battle%20system/Actors%20states/BattleCondition/Sticky.md)
- If we are instance.`inbattle`, the following happens on each of the player party members:
    - If `firepart` isn't null, it is destroyed
    - [BreakIce](../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the battleentity
    - `isasleep` is set to false
    - `isnumb` is set to false

#### `HPto1`

- If we aren't instance.`inbattle`, the `Damage0` sound is played
- The `hp` of the characterid player party member is set to 1

#### `GradualHP`

- If we are instance.`inbattle`
    - The `Heal3` sound is played
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called to apply the [GradualHP](../Battle%20system/Actors%20states/BattleCondition/GradualHP.md) condition on the characterid player party member for value amount of turn
    - If the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3
- Otherwise, an `HPRecover` is processed with the same characterid, but the value is doubled

#### `GradualHPParty`

- If we are instance.`inbattle`
    - The `Heal3` sound is played
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called to apply the [GradualHP](../Battle%20system/Actors%20states/BattleCondition/GradualHP.md) condition on each player party members that have an `hp` above 0 for value amount of turn
    - If the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3
- Otherwise, an `HPRecoverAll` is processed with the same characterid, but the value is doubled

#### `GradualTP`
- If we are instance.`inbattle`
    - The `Heal3` sound is played
    - [SetCondition](../Battle%20system/Actors%20states/Conditions%20methods/SetCondition.md) is called to apply the [GradualTP](../Battle%20system/Actors%20states/BattleCondition/GradualTP.md) condition on the characterid player party member for value amount of turn
    - If the `WeakStomach` [medal](../Enums%20and%20IDs/Medal.md) is equipped on the characterid player party member, an `AddPoison` effect is processed with the current value and characterid. The only difference is if `EternalPoison` isn's equipped, the amount of turn to apply the [Poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) condition is always 3
- Otherwise, an `TPRecover` is processed with the same characterid, but the value is doubled

## `Items` data

The asset contains one line per [Item](../Enums%20and%20IDs/Items.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the item|
|1|Unused description|[SetText](../SetText/SetText.md) string|An unused version of the item's description|
|2|Description|[SetText](../SetText/SetText.md) string|The description of the item|
|3|Prepender|[SetText](../SetText/SetText.md) string|The prepender string used with the [Anstring](../SetText/Individual%20commands/Anstring.md) command for this item|

The data will be loaded into `itemdata[0, id, x]` where `id` is the item id and `x` is the loaded index.

The unused description is, as its name says, not used in the game. It is safe to leave it empty or a dummy value, but its value must exist for the actual description which needs a valid value.

The prepender is optional: it will not be loaded if it does not exist. which leaves its spot in `itemdata` as null.
