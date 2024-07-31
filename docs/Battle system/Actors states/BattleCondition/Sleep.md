# Sleep
A stopping condition that will heal the attacker on each main turn and be removed on most attacks due to waking up (with caveats, see [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md) which can prevent waking up, halve the damage amount and heal for more per main turn.

## About `isasleep`
This condition feature a field that tells if it's present or not called `isasleep`. However, it's redundant: a [HasCondition](../Conditions%20methods/HasCondition.md) call gives the same information. It is nonetheless used in very specific edgecases instead of calling this method. The field is frequently updated to match the presence of the condition so in practice, this field can be ignored and treated as if a HasCondition returns above 0. Calling HasCondition also may set `isasleep` to true if it has the condition (only if it is the condition seeked or it occurs before the seeked one).

It is mentioned here because it remains important to make sure this field is updated if the condition's presence changes.

## Resistance
This condition has a dedicated resistance field for actors to use: `sleepres`. If it's 100 or above, the actor is immune to it. Inflictions that requires a resistance check will use this field.

## Resistance increases for player party members
For player party member, the resistance can only be increased by processing the `SleepRes` [BadgeEffects](../../../TextAsset%20Data/Medals%20data.md#medal-effects) with the value acting as the amount to increase it by. This only matters for enemy inflicting the player party member, it does not matter for user infliction using [items](../../../Enums%20and%20IDs/Items.md).

## Resistance increases for enemy party members
For player party member target when property is `Sleep` and when successfully inflicting this condition, [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md), target.`sleepres` is increased by 9. The increase is 13 instead if [HardMode](../../Damage%20pipeline/HardMode.md) returns true

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a [actimmobile](../Enemy%20features.md#actimmobile) is true). This include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

## [GetFreePlayerAmmount](../Player%20party%20members/GetFreePlayerAmmount.md)
This condition makes a player party member not count as free. This affects many logic such as knowing if a player can act.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition: 

- If the actor is an enemy party member, [isdefending](../Enemy%20features.md#isdefending) is set to false and the infliction overwrites the turn counter to the new one (meaning it won't stack)
- If the actor is a player party member, the infliction overwrites the turn counter if the new one is higher (meaning it won't stack, it can just reset it to a higher amount). An exception to this is when using an item that inflicted it which makes it stack

When inflicted as a new condition, if the actor is an enemy party member, its [isdefending](../Enemy%20features.md#isdefending) is set to false.

## [AddDelayedCondition](../Delayed%20condition.md)
This condition supports delayed infliction via AddDelayedCondition. Check its documentation to learn more.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it might be inflicted to the actor when called.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (`Relay` -> `SelectPlayer`)
This condition will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
This condition overrides block to be false meaning blocking is always denied for a player party member with this condition.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Sleep`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md).

This condition prevents toppling on enemy party members with a `ToppleFirst` or `ToppleAirOnly` [AttackProperty](../../Damage%20pipeline/AttackProperty.md) in their [weakness](../Enemy%20features.md#weakness).

In [DefaultDamageCalc](../../Damage%20pipeline/CalculateBaseDamage.md#defaultdamagecalc), this condition allows the `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md) to process by dividing the damage amount by 2 floored.

This condition my be removed during [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) or after its return in DoDamage damage calculation from a target with it after when sustaining damages under certain conditions (with caveats, check the methods's documentation to learn more). Notably, it won't happen if the target has the `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md). NOTE: If DoDamage decides to wake up the target, there is a known issue when setting their `cantmove` value. Check DoDamage's documentation to learn more.

## [EndPlayerTurn](../../Battle%20flow/EndPlayerTurn.md)
All enemy party members who still have this condition on EndPlayerTurn without [actimmobile](../Enemy%20features.md#actimmobile) will have their `cantmove` set to 1. NOTE: It means removing the condition on the same main turn it was inflicted may leave the `cantmove` at 1 even if the enemy party member should have been able to act during their phase.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0, but less than `maxhp`):

- A heal amount is determined with a starting value of the actor's `maxhp` * 0.1 ceiled. If the `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped on the actor, the amount is tripled. On top of this, if it's not a player party member, the amount is clamped from 0 to 4
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) with the amount determined earlier with a start of the actor position + its `cursoroffset` and an end of Vector3.up
- The actor's `hp` is incremented by the amount determined earlier clamped from 0 to the actor's `maxhp`
- The `Heal` sound is played
- A yield of 0.75 seconds is set to happen after the method is done
- The turn counter of the condition is decremented
- If the condition just expired, the actor's `cantmove` is set to 1 (this will become 0 if their `hp` is above 0 since turn advancement has yet to happen)
- `isasleep` is updated accordingly with the new status of the condition and if it's still true, the `cantmove` of the actor is set to 1 if it's an enemy party members and to 0 if it's a player party members. NOTE: It means removing the condition for enemy party members may leave the `cantmove` at 1 even if the enemy party member should have been able to act during their phase

## [RemoveCondition](../Conditions%20methods/RemoveCondition.md)
Removing this condition via this method will have the actor's `isasleep` set to false.

## GetBlock
For all alive (`hp` above 0 and not `dead`) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 14 (`Sleep`).

## [DoDamageAnim](../../Damage%20pipeline/DoDamage.md)
This condition may cause the actor's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) to be set to 14 (`Sleep`). Check the method's documentation for more details.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) may set to 14 (`Sleep`).

For all alive (`hp` above 0) enemy party members with this condition alongside [Flipped](Flipped.md) and whose `droproutine` isn't in progress, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 25 (`SleepFallen`) instead of 15 (`Fallen`). If they don't have [Flipped](Flipped.md), it may be set to 14 (`Sleep`).

Check the method's documentation to learn more.
