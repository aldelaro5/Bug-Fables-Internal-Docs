# RefreshSkills
This method allows to refresh the [skills](../Enums%20and%20IDs/Skills.md) list of all the player party members. The lists depends on [flags](../Flags%20arrays/flags.md), [medals](../Enums%20and%20IDs/Medal.md) equipped and others. It belongs to MainManager.

The method will go over each player party member and reset their `skills` to a new list before filling them with the ones they are allowed to have if the conditions are fufilled. Most of them are specific to each [animid](../Enums%20and%20IDs/AnimIDs.md) (only `Bee`, `Beetle` and `Moth` can have skills), but some of them are common with the same conditions to have it. 

The order they will be presented is the same order they will be checked and added if the condition is fufilled. The `Name` column is the name of the value in the `Skills` enum. To see the entire list of skills defined in the game by id order, check the [skills](../Enums%20and%20IDs/Skills.md) documentation.

## `Bee`'s skills

|Id|Name|Condition|
|-:|---------|------------|
|34|FieldBeemerang|instance.`battle` is null and flag 11 is true|
|35|FieldHalt|instance.`battle` is null and flag 21 is true|
|36|FieldFly|instance.`battle` is null and flag 19 is true|
|2|BeeRangMultiHit|None, this is always granted|
|18|HurricaneBeemerang|flag 21 is true|
|44|HeavyThrow|The `HeavyThrow` medal is equipped|
|16|NeedleToss|instance.`partylevel` is 11 or above|
|24|NeedlePincer|instance.`partylevel` is 15 or above|
|5|BeeFly|flags 19 is true|
|26|IceBeemerang|flag 544 is true|
|31|IceSphere|flag 533 is true|
|11|SecretStash|instance.`partylevel` is 2 or above|
|45|SharingStash|flag 445 is true|
|33|Sturdy|The `Tardigrade` is equipped on `Bee`|
|48|HardCharge|The `HardCharge` is equipped on `Bee`|
|46|RoyalDecree|`AntQueen` is a follower or the current [maps](../Enums%20and%20IDs/Maps.md) is `TestRoom` or flag 594 is true (During the fight near the refrigerator or the fire construct fight in chatper 7)|

## `Beetle`'s skills

|Id|Name|Condition|
|-:|---------|------------|
|37|FieldHorn|instance.`battle` is null and flag 17 is true|
|49|FirstDash|instance.`battle` is null, flag 699 is true and flag 39 is false|
|38|FieldDash|instance.`battle` is null and flag 39 is true|
|39|FieldDig|instance.`battle` is null and flag 18 is true|
|3|BeetleTaunt|None, this is always granted|
|32|HeavyStrike|instance.`partylevel` is 4 or above|
|6|BeetleDig|flag 18 is true|
|10|HornDash|flag 699 is true|
|5|BeeFly|flag 19 is true|
|27|IceDrill|flag 17 is true|
|31|IceSphere|flag 533 is true|
|9|PeebleToss|The `MightyPeeble` medal is equipped|
|19|PebbleTossPlus|The `MightierPebble` medal is equipped|
|43|PepTalk|flag 98 is true|
|33|Sturdy|The `Tardigrade` medal is equipped on `Beetle`|
|48|HardCharge|The `HardCharge` medal is equipped on `Beetle`|
|46|RoyalDecree|`AntQueen` is a follower or the current [maps](../Enums%20and%20IDs/Maps.md) is `TestRoom` or flag 594 is true (During the fight near the refrigerator or the fire construct fight in chatper 7)|

## `Moth`'s skills

|Id|Name|Condition|
|-:|---------|------------|
|40|FieldFreeze|instance.`battle` is null|
|42|FieldShield|instance.`battle` is null and flag 20 is true|
|41|FieldIcecle|instance.`battle` is null and flag 171 is true|
|4|Icefall|None, this is always granted|
|21|FrigidCoffin|instance.`partylevel` is 6 or above|
|25|IceRain|flag 171 is true|
|27|IceDrill|flag 17 is true|
|26|IceBeemerang|flag 544 is true|
|31|IceSphere|flag 533 is true|
|17|BubbleShieldLite|flag 160 is true|
|7|BubbleShield|flag 20 is true|
|8|Empower|The `ChargeUp` medal is equipped|
|22|ChargeUpPlus|The `ChargeUp2` medal is equipped|
|14|AttackUp|The `Empower` medal is equipped|
|23|EmpowerPlus|The `Empower2` medal is equipped|
|15|DefenseUp|The `Fortify` medal is equipped|
|30|DefenseUpPlus|The `Fortify2` medal is equipped|
|28|AttackDown|The `Emfeeble` medal is equipped|
|29|AttackDownPlus|The `Emfeeble2` medal is equipped|
|12|DefenseBreak1|The `Depower1` medal is equipped|
|13|DefenseBreakAll|The `Depower2` medal is equipped|
|47|Cleanse|instance.`partylevel` is 13 or above|
|33|Sturdy|The `Tardigrade` medal is equipped on `Moth`|
|48|HardCharge|The `HardCharge` medal is equipped on `Moth`|
|46|RoyalDecree|`AntQueen` is a follower or the current [maps](../Enums%20and%20IDs/Maps.md) is `TestRoom` or flag 594 is true (During the fight near the refrigerator or the fire construct fight in chatper 7)|
