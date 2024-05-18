# `NeedlePincer`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{random between 4 and 6 include (Confirm, Cancel or Switch)}
|2|DoCommand 1 set `commandsuccess` to true|45.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{random between 4 and 6 include (Confirm, Cancel or Switch)}
|3|`Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped and DoCommand 2 set `commandsuccess` to true|45.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{random between 4 and 6 include (Confirm, Cancel or Switch)}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1a|DoCommand 1 set `commandsuccess` to true|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` unclamped|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|1b|DoCommand 1 set `commandsuccess` to false|`playerdata[currentturn]`|The enemy party member|attacker's `atk` clamped from -99 to 1|null|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|
|2a|DoCommand 2 set `commandsuccess` to true|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` * 0.7 floored clamped from 1 to 99. NOTE: This clamp incorrectly removes the influences of negative `atk` values|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|2b|DoCommand 2 set `commandsuccess` to false|`playerdata[currentturn]`|The enemy party member|attacker's `atk` clamped from -99 to 1|null|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|
|3a|`Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped and DoCommand 3 set `commandsuccess` to true|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` * 0.49 floored clamped from 1 to 99. NOTE: This clamp incorrectly removes the influences of negative `atk` values|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|3b|`Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped and DoCommand 3 set `commandsuccess` to false|`playerdata[currentturn]`|The enemy party member|attacker's `atk` clamped from -99 to 1|null|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|

## PincerStatus
Every DoDamage call after `commandsuccess` was set to true from the last DoCommand call has [Pierce](../../Damage%20pipeline/AttackProperty.md) meaning they can't inflict any [conditions](../../Actors%20states/Conditions.md) by themselves. However, after each DoDamage call (if applicable), a method called PincerStatus happens which is exclusive to this action and [NeedlePincer](NeedlePincer.md). It implies custom condition infliction logic.

What's unique about this infliction logic is multiple conditions could be inflicted at once and each requires at least having a [medal](../../../Enums%20and%20IDs/Medal.md) equipped for the infliction attempt to occur. Here are the possible conditions and their requirements (not mutually exclusive, checked and attempted in the order presented):

- If `PoisonNeedle` medal is equipped, 2 turns of [Poison](../../Actors%20states/BattleCondition/Poison.md)
- If `ElecNeedles` medal is equipped, 1 turn of [Numb](../../Actors%20states/BattleCondition/Numb.md)
- If `NumbNeedle` medal is equipped and the enemy party member doesn't have the [Numb](../../Actors%20states/BattleCondition/Numb.md) condition, 2 turns of [Sleep](../../Actors%20states/BattleCondition/Sleep.md)

However, the way the infliction attempt is done isn't via [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md), but via a method only used by PincerStatus called [TryCondition](../../Actors%20states/Conditions%20methods/TryConditions.md) which behaves similarily to [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md).

> NOTE: Although similar, [TryCondition](../../Actors%20states/Conditions%20methods/TryConditions.md) has many differences depending on the condition compared to [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) and it even has an off by one error in the resistance check. Check its documentation to learn more

## Logic Sequence

- attacker animstate set to 115
- Yield for 0.65 seconds
- The camera moves slowely up and back
- attacker MoveTowards in front of the target (offset in x by its size) at 2.0 speed stopping at animstate 121
- Yield until the attacker's `forcemove` is done
- DoCommand 1 call happens
- Yield for a frame
- Yield until `doingaction` goes to false
- If `commandsuccess` is false:
    - `AtkFail` sound plays if it wasn't already playing
    - attacker animstate set to 106
    - Yield for 0.25 seconds
    - DoDamage 1b happens
- Otherwise (`commandsuccess` is true):
    - attacker MoveTowards 0.65 in x over 10.0 frames then back to its original position (MainManager coroutine)
    - DoDamage 1a happens
    - PincerStatus is called (check the section above for more details)
    - attacker animstate set to 122
    - Yield until `commandsprite[0]` goes to null (this means not only the action command is done, but that FinishAction's destroy calls are done)
    - Yield for a frame
    - Second and third hit loop (only one iteration unless `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped where it's 2 iterations):
        - DoCommand 2 (or 3) happens
        - Yield for a frame
        - Yield until `doingaction` goes to false
        - If `commandsuccess` is false, the same logic done than the first hit fail mentioned above is done (with DoDamage 2b or 3b) and break out of the loop early
        - attacker MoveTowards 0.65 in x over 10.0 frames then back to its original position (MainManager coroutine)
        - DoDamage 2a (or 3a) happens
        - PincerStatus is called (check the section above for more details)
        - attacker animstate set to 123
        - Break out of the loop if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is not equipped or if it's the second time this is reached
        - Yield until `commandsprite[0]` goes to null (this means not only the action command is done, but that FinishAction's destroy calls are done)
        - Yield for a frame
        - attacker animsate set to 121
- Yield for 0.5 seconds