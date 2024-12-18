# AttackProperty
An AttackProperty is a parameter of [DoDamage](DoDamage.md) and they are also the elements allowed in the [weakness](../Actors%20states/Enemy%20features.md#weakness) field of [BattleData](../Actors%20states/BattleData.md) (meant for enemy party members only). They influence the damage pipeline in various ways either at the specific call level when passed as a parameter to DoDamage or at the target level for an enemy party member's `weakness`. They mainly influence [CalculateBaseDamage](CalculateBaseDamage.md)'s logic.

They are not to be confused with [DamageOverride](DamageOverride.md) which can only be sent to DoDamage as an array of them and have very different logic changes.

Here are the different AttackProperty, how they are meant to be passed to the damage pipeline and a summary of their effects.

|Value|Name|DoDamage parameter?|`weakness`?|Description|
|-----:|----|-------------------|-----------|-----------|
|0|Pierce|Yes|No|Ignores all calculations related to defense in CalculateBaseDamage. Only valid for enemy party member targets that do not have `AntiPierce` in their `weakness`|
|1|Flip|Yes|Yes|When used as a DoDamage parameter, 1 point of defense in CalculateBaseDamage will be ignored when the target doesn't have the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition (this is done incorrectly, see the CalculateBaseDamage documentation to learn more). It will also attempt to inflict the `Flipped` condition if the target has this property in its `weakness` and some other conditions are fufilled. It will also bypass [defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) and prevent `isdefending` from being set to true (it can even break the guard)|
|2|Freeze|Yes|No|Attempts to inflict the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition during CalculateBaseDamage|
|3|Poison|Yes|No|Attempts to inflict the [Poison](../Actors%20states/BattleCondition/Poison.md) condition during CalculateBaseDamage|
|4|Numb|Yes|No|Attempts to inflict the [Numb](../Actors%20states/BattleCondition/Numb.md) condition during CalculateBaseDamage|
|5|Sleep|Yes|No|Attempts to inflict the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition during CalculateBaseDamage|
|6|AntiPierce|No|Yes|Prevents the `Pierce` property to have any effect during CalculateBaseDamage (only the `Spuder` [enemy](../../Enums%20and%20IDs/Enemies.md) can get it via [CheckEvent](../Battle%20flow/Update.md) when applicable)|
|7|ToppleFirst|No|Yes|The enemy party member will receive the [Topple](../Actors%20states/BattleCondition/Topple.md) condition when it is damaged first before falling or getting the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition during CalculateBaseDamage. It acts as defensive mechanism to enemy party members where they would have fell or flipped immediately if they didn't feature this `weakness`|
|8|Magic|Yes|Yes|When used an enemy party member's `weakness`, it will allow a magic weakness to be processed in CalculateBaseDamage when property is also `Magic` or when the attacker is a player party member while `lastskill` is `Icefall`, `FrigidCoffin` or `IceRain` [skill](../../Enums%20and%20IDs/Skills.md) alongside some other conditions|
|9|ToppleAirOnly|No|Yes|The same as `ToppleFirst`, but only applicable when the enemy party member's [position](../Actors%20states/BattlePosition.md) is `Flying`|
|10|Atleast1|Yes|No|Clamps the damage amount from 1 towards the end of CalculateBaseDamage unless the target has `LimitX10` in its `weakness` where 10 is subtracted from the amount instead|
|11|Atleast1pierce|Yes|No|Combines the effects of `Pierce` and `Atleast1`|
|12|LimitX10|No|Yes|Gives ? defense logic to the enemy party member which CalculateBaseDamage processes towards the end where the amount is first subtracted by 10 if property is `Atleast1` or `Atleast1pierce` and then set to 0 if it was <= 1 or divided by 10 ceiled if it was above 1|
|13|NoExceptions|Yes|No|Ignores many, but not all logic in CalculateBaseDamage. If the target is invulnerable according to DoDamage, the damage dealt will be the ammount sent|
|14|None|Yes|No|No property applies, the same as sending null. DoDamage will override this to null when passing it to CalculateBaseDamage|
|15|HornExtraDamage|No|Yes|Process a +1 damage bonus when dealing damages to the enemy party member if property is `Flip` with weaknesshit|
|16|MinusMagic|No|No|UNUSED|
|17|DieInOneHit|No|Yes|The enemy party member's `hp` is set to 0 on DoDamage, killing him in one hit|
|18|SurviveWith10|No|Yes|The final target's `hp` is clamped from 10 except if `lastskill` is 9 (`PeebleToss`) [skill](../../Enums%20and%20IDs/Skills.md)|
|19|FlyOnFlip|No|Yes|The enemy party member's [position](../Actors%20states/BattlePosition.md) is changed to `Flying` if the property of the attack is `Flip` and the enemy party member's `position` was `Ground`|
|20|Fire|Yes|No|Attempts to inflict the [Fire](../Actors%20states/BattleCondition/Fire.md) condition during CalculateBaseDamage|
|21|MultiHitDefense|No|No|UNUSED|
|22|DefDownOnBlock|Yes|No|Inflicts the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition on the target when they are still alive and don't have the [Shield](../Actors%20states/BattleCondition/Shield.md) condition|
|23|DefDownOnFlyHard|No|Yes|Grant a +1 damage bonus if the target's [position](../Actors%20states/BattlePosition.md) is `Flying` and [HardMode](HardMode.md) returns true (This is specific to a `VenusBoss` [enemy](../../Enums%20and%20IDs/Enemies.md#enemies))|
|24|Ink|Yes|No|Attempts to inflict the [Inked](../Actors%20states/BattleCondition/Inked.md) condition during CalculateBaseDamage|
|25|AtkDownOnBlock|Yes|No|Inflicts the [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) condition on the target when they are still alive and don't have the [Shield](../Actors%20states/BattleCondition/Shield.md) condition|
|26|Sticky|Yes|No|Attempts to inflict the [Sticky](../Actors%20states/BattleCondition/Sticky.md) condition during CalculateBaseDamage|
|27|InkOnBlock|Yes|No|The same then `Ink`, but the infliction can still occur even if the player sucessfully blocked|
|28|AlwaysSurvive|No|Yes|The final target's `hp` is clamped from 1|
|29|Numb1Turn|Yes|No|The same as `Numb`, but the infliction is always for 1 turn regardless if it's a `chompyaction` or not|
|30|DoBlock|No|Yes|If the the enemy party member isn't IsStoppedLite (same than [IsStopped](../Actors%20states/IsStopped.md), but [Flipped](../Actors%20states/BattleCondition/Flipped.md) doesn't count as being stopped), the enemy party member will simulate a block by reducing the damages for the second DoDamage call in any actor turn by dividing the amount after [CalculateBaseDamage](CalculateBaseDamage.md) by 2 floored and on any further DoDamage calls in the same actor turn, the amount will be divided by 3 floored (the amount of DoDamage calls is tracked by their `blockTimes`)|
|31|Holographic|No|Yes|This will force the enemy party member's battle entity to have its `hologram` set to true on [StartBattle](../StartBattle.md)|
|32|PoisonDamUp|No|Yes|If the enemy party member has the [poison](../Actors%20states/BattleCondition/Poison.md) condition and they are the attacker of a DoDamage call, 2 damage will be added to the total after [CalculateBaseDamage](CalculateBaseDamage.md)|
|33|Raw|Yes|No|This will cause [CalculateBaseDamage](CalculateBaseDamage.md) to discard any effects on the damage amount with the exception of player blocking effects. It is only meant to be used for enemy party members's action logic|
