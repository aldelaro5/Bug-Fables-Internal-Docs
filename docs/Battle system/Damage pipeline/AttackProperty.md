# AttackProperty
An AttackProperty is a parameter of [DoDamage](DoDamage.md) and they are also the elements allowed in the `weakness` field of BattleData (meant for enemy party members only). They influence the damage pipeline in various ways either at the specific call level when passed as a parameter to DoDamage or at the target level for an enemy party member's `weakness`. They mainly influence [CalculateBaseDamage](CalculateBaseDamage.md)'s logic.

Here are the different AttackProperty, how they are meant to be passed to the damage pipeline and a summary of their effects.

|Value|Name|DoDamage parameter?|`weakness`?|Description|
|-----|----|-------------------|-----------|-----------|
|0|Pierce|Yes|No|Ignores all calculations related to defense in CalculateBaseDamage. Only valid for enemy party member targets that do not have `AntiPierce` in their `weakness`|
|1|Flip|Yes|Yes|When used as a DoDamage parameter, 1 point of defense in CalculateBaseDamage will be ignored when the target doesn't have the `Flipped` condition (this is done incorrectly, see the CalculateBaseDamage documentation to learn more). It will also attempt to inflict the `Flipped` condition if the target has this property in its `weakness` and some other conditions are fufilled|
|2|Freeze|Yes|No|Attempts to inflict the `Freeze` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|3|Poison|Yes|No|Attempts to inflict the `Poison` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|4|Numb|Yes|No|Attempts to inflict the `Numb` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|5|Sleep|Yes|No|Attempts to inflict the `Sleep` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|6|AntiPierce|No|Yes|Prevents the `Pierce` property to have any effect during CalculateBaseDamage (only the `Spuder` [enemy](../../Enums%20and%20IDs/Enemies.md) can get it via [CheckEvent](../Battle%20flow/Update.md) when applicable)|
|7|ToppleFirst|No|Yes|The enemy party member will receive the `Topple` [condition](../Actors%20states/Conditions.md) when it is damaged first before falling or getting the `Flipped` condition during CalculateBaseDamage. It acts as defensive mechanism to enemy party members where they would have fell or flipped immediately if they didn't feature this `weakness`|
|8|Magic|Yes|Yes|When used an enemy party member's `weakness`, it will allow a magic weakness to be processed in CalculateBaseDamage when property is also `Magic` or when the attacker is a player party member while `lastskill` is `Icefall`, `FrigidCoffin` or `IceRain` alongside some other conditions|
|9|ToppleAirOnly|No|Yes|The same as `ToppleFirst`, but only applicable when the enemy party member's `position` is `Flying`|
|10|Atleast1|Yes|No|Clamps the damage amount from 1 towards the end of CalculateBaseDamage unless the target has `LimitX10` in its `weakness` where 10 is subtracted from the amount instead|
|11|Atleast1pierce|Yes|No|Combines the effects of `Pierce` and `Atleast1pierce`|
|12|LimitX10|No|Yes|Gives ? defense logic to the enemy party member which CalculateBaseDamage processes towards the end where the amount is first subtracted by 10 if property is `Atleast1` or `Atleast1pierce` and then set to 0 if it was <= 1 or divided by 10 ceiled if it was above 1|
|13|NoExceptions|Yes|No|Ignores many, but not all logic in CalculateBaseDamage. If the target is invulnerable according to DoDamage, the damage dealt will be the one sent in|
|14|None|Yes|No|No property applies, the same as sensing null. DoDamage will override this to null when passing it to CalculateBaseDamage|
|15|HornExtraDamage|No|Yes|Process a +1 damage bonus when dealing damages to the enemy party member if property is `Flip` with weaknesshit|
|16|MinusMagic|No|No|UNUSED|
|17|DieInOneHit|No|Yes|The enemy party member's `hp` is set to 0 on DoDamage, killing him in one hit|
|18|SurviveWith10|No|Yes|The final target's `hp` is clamped from 10 except if `lastskill` is 9 (`PeebleToss`)|
|19|FlyOnFlip|No|Yes|The enemy party member's `position` is changed to `Flying` if the property of the attack is `Flip` and the enemy party member's `position` was `Ground`|
|20|Fire|Yes|No|Attempts to inflict the `Fire` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|21|MultiHitDefense|No|No|UNUSED|
|22|DefDownOnBlock|Yes|No|Inflicts the `DefenseDown` [condition](../Actors%20states/Conditions.md) on the target when they are still alive and don't have the `Shield` condition|
|23|DefDownOnFlyHard|No|Yes|Grant a +1 damage bonus if the target is `Flying` and HardMode returns true (This is specific to a `VenusBoss` [enemy](../../Enums%20and%20IDs/Enemies.md#enemies))|
|24|Ink|Yes|No|Attempts to inflict the `Inked` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|25|AtkDownOnBlock|Yes|No|Inflicts the `AttackDown` [condition](../Actors%20states/Conditions.md) on the target when they are still alive and don't have the `Shield` condition|
|26|Sticky|Yes|No|Attempts to inflict the `Sticky` [condition](../Actors%20states/Conditions.md) during CalculateBaseDamage|
|27|InkOnBlock|Yes|No|The same then `Ink`, but the infliction can still occur even if the player sucessfully blocked|
|28|AlwaysSurvive|No|Yes|The final target's `hp` is clamped from 1|
|29|Numb1Turn|Yes|No|The same as `Numb`, but the infliction is always for 1 turn regardless if it's a `chompyaction` or not|