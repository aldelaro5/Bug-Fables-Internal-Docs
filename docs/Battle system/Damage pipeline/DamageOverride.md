# DamageOverride
A DamageOverride is a way to modify some logic in the damage pipeline. Unlike [AttackProperty](AttackProperty.md), they can only be sent to [DoDamage](DoDamage.md) in a parameter, but that parameter takes an array meaning it is possible to send multiple DamageOverrides (some however are mutually exclusive against each other).

The order in which they are specified in the parameter matters as they are processed sequentially.

The vast majority are processed in DoDamage, but some are in [CalculateBaseDamage](CalculateBaseDamage.md) and they can influence the logic there.

|Value|Name|Description|
|-----|----|-----------|
|0|LeftMovement|UNUSED|
|1|RightMovement|UNUSED|
|2|UpMovement|UNUSED|
|3|NoMovement|UNUSED|
|4|HurtStyle|UNUSED|
|5|HealStyle|UNUSED|
|6|TPStyle|UNUSED|
|7|MAXStyle|UNUSED|
|8|ShakeLonger|UNUSED|
|9|NoShake|UNUSED|
|10|ShowCombo|Calls [ShowComboMessage](../Visual%20rendering/ShowSuccessWord.md#showcombomessage) right after the damage amount has been calculated in DoDamage|
|11|NoSound|The [DoDamageAnim](DoDamageAnim.md) call towards the end of DoDamage will have nosound set to true. NOTE: this is ignored if `NoDamageAnim` or `BlockSoundOnly` is processed after|
|12|FailSound|The [DoDamageAnim](DoDamageAnim.md) call towards the end of DoDamage will have fail set to true. NOTE: this is ignored if `NoDamageAnim` is processed after|
|13|NoDamageAnim|There will not be a [DoDamageAnim](DoDamageAnim.md) call towards the end of DoDamage. NOTE: processing this overrides any `NoSound`, `BlockSoundOnly`, `FailSound` and `FakeAnim` because the call won't happen|
|14|BlockSoundOnly|The [DoDamageAnim](DoDamageAnim.md) call towards the end of DoDamage will have nosound set to true if block is false. NOTE: this is ignored if `NoDamageAnim` or `NoSound` is processed after|
|15|NoCounter|There will not be a [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) call in DoDamage (this does not cover a `Frostbite` [medal](../../Enums%20and%20IDs/Medal.md) counter which is done in CalculateBaseDamage)|
|16|NoFall|Prevents an enemy party member from falling under any circumstances in CalculateBaseDamage if one is the target|
|17|NoIceBreak|Prevents an enemy party member from thawing out when they have the `Freeze` [condition](../Actors%20states/Conditions.md) under any circumstances in CalculateBaseDamage if one is the target|
|18|FakeAnim|The [DoDamageAnim](DoDamageAnim.md) call towards the end of DoDamage will have fakeanim set to true. NOTE: this is ignored if `NoDamageAnim` is processed after|
|19|DontAwake|Prevents the attack from removing the `Sleep` [condition](../Actors%20states/Conditions.md) on the target during the course of DoDamage. NOTE: CalculateBaseDamage can still remove it when inflicting a different status, check its documentation to learn more|
|20|IgnoreNumb|Prevents the `Numb` [condition](../Actors%20states/Conditions.md) to have its defense apply during CalculateBaseDamage under any circumstances|
