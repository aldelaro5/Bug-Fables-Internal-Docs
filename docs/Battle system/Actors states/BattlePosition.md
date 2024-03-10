# BattlePosition
This is an enum that influences the ability for an enemy party member to be targetted in battle. It can be thought of as the logical position of the enemy party member.

On [BattleData](BattleData.md), the position of the enemy party member is stored in the `position` field.

|Value|Name|Description|
|-----|----|-----------|
|0|Ground|The enemy party member is grounded and a [GetAvaliableTargets](Targetting/GetAvaliableTargets.md) call with onlyground to true will include the target. This is also the position when the battleentity.`height` isn't above `minheight` + 0.5|
|1|Flying|The enemy party member isn't grounded and a [GetAvaliableTargets](Targetting/GetAvaliableTargets.md) call with onlyground to true will exclude the target. This is also the position when the battleentity.`height` is above `minheight` + 0.5. Depending on many factors such as [cantfall](Enemy%20features.md#cantfall) or having a topple [weakness](Enemy%20features.md#weakness), the enemy party member may be subject to a [Drop](../../Entities/EntityControl/Notable%20methods/Drop.md) during [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md)|
|2|OutOfReach|The enemy party member cannot be targeted and [GetAvaliableTargets](Targetting/GetAvaliableTargets.md) will always exclude it|
|3|Random|A value only used during [GetEnemyData](../../TextAsset%20Data/Enemies%20data.md#getenemydata) where the position will get changed to `Ground` or `Flying` so this value is never used in battle, only during data loading|
|4|Underground|The enemy party member is underground which imposes restrictions for the target to be included in [GetAvaliableTargets](Targetting/GetAvaliableTargets.md) and for using [items](../../Enums%20and%20IDs/Items.md)|
