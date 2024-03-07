# Freeze
A stopping condition that encases the actor in a block of ice. Most attacks will remove this condition and thaw out the ice, but with a +1 damage to the attack. It also allows the target to process a `FrostBite` [medal](../../../Enums%20and%20IDs/Medal.md) when attacked.

## Resistance
This condition has a dedicated resistance field for actors to use: `freezeres`. If it's 100 or above, the actor is immune to it. Inflictions that requires a resistance check will use this field.

## Resistance increases for player party members
For player party member, the resistance can only be increased by processing the `FreezeRes` [BadgeEffects](../../../TextAsset%20Data/Medals%20data.md#medal-effects) with the value acting as the amount to increase it by. This only matters for enemy inflicting the player party member, it does not matter for user infliction using [items](../../../Enums%20and%20IDs/Items.md).

## Resistance increases for enemy party members
Here are the potential increases for the resistance associated (only applies to enemy party members):

- On [StartBattle](../../StartBattle.md) (not mutually exclusive):
    - If the calledfrom.entity or the battleentity is `inice` while it's a `Krawler` or `CursedSkull` [enemy](../../../Enums%20and%20IDs/Enemies.md), the enemy party member's `freezeres` is increased by 70
    - If battleentity.`forcefire` or we are in the `GiantLair` [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../../../Enums%20and%20IDs/Maps.md) while the enemy is a `Krawler`, `CursedSkull` or `Cape`, the enemy party member's `freezeres` is set to 110 making them immune (this override the clause above if it applied)
- On [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) when sucessfully inflicting this condition when property is `Freeze` (mutually exclusive, only the first that applies):
    - If the corresponding [endata](../../../TextAsset%20Data/Entity%20data.md#entity-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` area, target.`freezeres` is increased by 70
    - Otherwise, if target.`frozenlastturn` is true, target.`freezeres` is increased by 25
    - Otherwise, target.`freezeres` is increased by 13. The increase is 18 instead if [HardMode](../../Damage%20pipeline/HardMode.md) returns true

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a `actimmobile` is true). This include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

## [GetFreePlayerAmmount](../Player%20party%20members/GetFreePlayerAmmount.md)
This condition makes a player party member not count as free. This affects many logic such as knowing if a player can act.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition: 

- If the actor is an enemy party member, `isdefending` is set to false and the infliction overwrites the turn counter to the new one (meaning it won't stack)
- If the actor is a player party member, the infliction overwrites the turn counter if the new one is higher (meaning it won't stack, it can just reset it to a higher amount). An exception to this is when using an item that inflicted it which makes it stack

When inflicted as a new condition, if the actor is an enemy party member, its `isdefending` is set to false.

## [AddDelayedCondition](../Delayed%20condition.md)
This condition supports delayed infliction via AddDelayedCondition. Check its documentation to learn more.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it might be inflicted to the actor when called.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (`Relay` -> `SelectPlayer`)
This condition will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
This condition overrides block to be false meaning blocking is always denied for a player party member with this condition.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Freeze`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md).

This condition prevents toppling on enemy party members with a `ToppleFirst` or `ToppleAirOnly` [AttackProperty](../../Damage%20pipeline/AttackProperty.md) in their `weakness`.

This condition also implicates entire logic related to it unless the `NoIceBreak` [override](../../Damage%20pipeline/DamageOverride.md) is present. Check the CalculateBaseDamage documentation to learn more, but in summary:

- Special calculation logic when the target has the `FrostBite` [medal](../../../Enums%20and%20IDs/Medal.md)
- A +1 damage when `FrostBite` didn't apply followed by the removal of this condition alongside some other ice thawing logic

## [EndPlayerTurn](../../Battle%20flow/EndPlayerTurn.md)
All enemy party members who still have this condition on EndPlayerTurn without `actimmobile` will have their `cantmove` set to 1. NOTE: It means removing the condition on the same main turn it was inflicted may leave the `cantmove` at 1 even if the enemy party member should have been able to act during their phase.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0):

- The `Freeze` sound is played on the battleentity at 1.5 pitch
- battleentity.`shakeice` is set to true
- A yield of 0.75 seconds is set to happen after the method is done
- The turn counter of the condition is decremented
- If the turn counter still hasn't reached 0:
    - `frozenlastturn` is set to true which affects [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) if property is `Freeze` since it will increase target.`freezeres` by 25 instead of 13 (18 when HardMode returns true). NOTE: while the `FrigidCoffin` [skill](../../../Enums%20and%20IDs/Skills.md) inflicts this condition outside of the damage pipeline, it still explicitly follow this logic and is therefore affected the same way
    - The `cantmove` of the actor is set to 1 if it's an enemy party members and to 0 if it's a player party members. NOTE: It means removing the condition for enemy party members may leave the `cantmove` at 1 even if the enemy party member should have been able to act during their phase
- Otherwise (the condition just expired)
    - `frozenlastturn` is set to false
    - [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) is called on the actor's battleentity if its `icecube` exists
    - The actor's `cantmove` is set to 1 (this will become 0 if their `hp` is above 0 since turn advancement has yet to happen)

## [RemoveCondition](../Conditions%20methods/RemoveCondition.md)
Removing this condition via this method will have [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) called on the entity if it exists (not the battleentity).

## [HealConditions](../Conditions%20methods/HealConditions.md)
This condition is supported by this method and it will be removed when called.

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
When this method is called while the actor has this condition, [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) is called on the battleentity.

## GetBlock
For all alive (`hp` above 0 and not `dead`) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`).

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`).

## [AddExperience](../../Battle%20flow/Terminal%20coroutines/AddExperience.md)
All player party members who still had this condition when AddExperience occurs will have [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) called on their battleentity.

## GetEnemySize
This condition causes this method to return the enemy party member's `freezesize` if it is above 0.1 instead of its `size`. This can affect the following [skills](../../../Enums%20and%20IDs/Skills.md) and  player party members' attacks where the attacker will move to a slightly different position towards the target during the action:

- `firststrike` action (when the [animid](../../../Enums%20and%20IDs/AnimIDs.md) is `Beetle`)
- `Beetle`'s basic attack (when the [animid](../../../Enums%20and%20IDs/AnimIDs.md) is `Beetle`)
- `HeavyStrike`
- `NeedlePincer`

## [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop)
The entity will drop to 0.0 instead of its `minheight` when it has this condition.
