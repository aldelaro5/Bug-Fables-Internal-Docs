# Numb
A stopping condition that gives the actor a point of defense (with caveats, see [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `ShockTrooper` [medal](../../../Enums%20and%20IDs/Medal.md) which can bypass damage calculation.

## About `isnumb`
This condition feature a field that tells if it's present or not called `isnumb`. However, it's redundant: a [HasCondition](../Conditions%20methods/HasCondition.md) call gives the same information. It is nonetheless used in very specific edgecases instead of calling this method. The field is frequently updated to match the presence of the condition so in practice, this field can be ignored and treated as if a HasCondition returns above 0.

It is mentioned here because it remains important to make sure this field is updated if the condition's presence changes.

## Resistance
This condition has a dedicated resistance field for actors to use: `numbres`. If it's 100 or above, the actor is immune to it. Inflictions that requires a resistance check will use this field.

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a `actimmobile` is true). This include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

## [GetFreePlayerAmmount](../Player%20party%20members/GetFreePlayerAmmount.md)
This condition makes a player party member not count as free. This affects many logic such as knowing if a player can act.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition: 

- If the actor is an enemy party member, `isdefending` is set to false and the infliction overwrites the turn counter to the new one (meaning it won't stack)
- If the actor is a player party member, the infliction overwrites the turn counter if the new one is higher (meaning it won't stack, it can just reset it to a higher amount). An exception to this is when using an item that inflicted it which makes it stack

When inflicted as a new condition, if the actor is an enemy party member, its `isdefending` is set to false.

Any method call updates `isnumb` accordingly.

## [AddDelayedCondition](../Delayed%20condition.md)
This condition supports delayed infliction via AddDelayedCondition. Check its documentation to learn more.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it might be inflicted to the actor when called.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (`Relay` -> `SelectPlayer`)
This condition will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
This condition overrides block to be false meaning blocking is always denied for a player party member with this condition.

This condition allows a target with a `ShockTrooper` [medal](../../../Enums%20and%20IDs/Medal.md) to be considered Invulnerable and bypass damage calculation completely. In such case, the damage ammount is overriden to 0 unless property is `NoExceptions` where it's overriden by the original damageammount sent to the method.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Numb` or `Numb1Turn` (it's the same as `Numb`, but it always inflict for 1 turn). This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md).

This condition prevents toppling on enemy party members with a `ToppleFirst` or `ToppleAirOnly` [AttackProperty](../../Damage%20pipeline/AttackProperty.md) in their `weakness`.

This condition gives a point of defense recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#TrueDef) and will result in a -1 damage. NOTE: this has several caveats, check CalculateBaseDamage's documentation to learn more.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0):

- The `ElecFast` particles are played at the battleentity position + Vector3.up
- A ShakeSprite coroutine is started on the battleentity with the intensity being (0.1, 0.15, 0.0) for 40.0 frames
- The `Shock` sound is played on the battleentity at 1.25 pitch
- A yield of 0.75 seconds is set to happen after the method is done
- The turn counter of the condition is decremented
- If the condition just expired, the actor's `cantmove` is set to 1 (this will become 0 if their `hp` is above 0 since turn advancement has yet to happen)
- `isnumb` is updated accordingly with the new status of the condition and if it's still true, the `cantmove` of the actor is set to 1 if it's an enemy party members and to 0 if it's a player party members. NOTE: It means removing the condition for enemy party members may leave the `cantmove` at 1 even if the enemy party member should have been able to act during their phase

## [RemoveCondition](../Conditions%20methods/RemoveCondition.md)
Removing this condition via this method will have the actor's `isnumb` set to false.

## [HealConditions](../Conditions%20methods/HealConditions.md)
This condition is supported by this method and it will be removed when called on top of setting the actor's `isnumb` to false.

## GetBlock
For all alive (`hp` above 0 and not `dead`) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`).

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`).

## EntityControl.[LateUpdate](../../../Entities/EntityControl/Update%20process/Unity%20events/LateUpdate.md)
For any battle entities that isn't `dead` or `iskill` while not in a [terminal flow](../../Battle%20flow/Update%20flows/Terminal%20flow.md), [Numb](../../../Entities/EntityControl/EntityControl%20Methods.md#numb) is called if `isnumb` is true which plays the periodic numbing visual effects.