# TryCondition
A method that is only used by NeedlerPincer to attempt to inflict conditions as part of the [NeedleToss](../../Player%20actions/Skills/NeedleToss.md) and [NeedlePincer](../../Player%20actions/Skills/NeedlePincer.md) actions (with caveats and dead logic, see below to learn more).

It functions similarily than the infliction logic of  [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md), but it has several differences (even new issues) and it doesn't support all conditions that the damage pipeline supports.

```cs
private void TryCondition(ref MainManager.BattleData data, MainManager.BattleCondition condition, int turns)
```

## Supported conditions
It only supports the following [conditions](../Conditions.md) (the method does nothing for any other):

- [Freeze](../BattleCondition/Freeze.md) (never used under normal gameplay), requires passing a resistance check with the target's `freezeres` for infliction
- [Fire](../BattleCondition/Fire.md) (never used under normal gameplay), requires the same conditions then [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md), but without the block check or [Sturdy](../BattleCondition/Sturdy.md) check for infliction
- [Sleep](../BattleCondition/Sleep.md), requires passing a resistance check with the target's `sleepres` for infliction
- [Numb](../BattleCondition/Numb.md), requires passing a resistance check with the target's `numbres` for infliction
- [Poison](../BattleCondition/Poison.md), requires passing a resistance check with the target's `poisonres` for infliction

## Resistance check differences
The resistance check does have logic to take into account player infliction including logic to reduce the corresponding resistance by 15 if the `StatusBoost` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped on the target, but this logic is never used under normal gameplay. This is because the method is only used when the player attacks an enemy and in these cases, the enemy is the target.

## Issue with the resistance checks
> NOTE: All these resistance checks are off by 1. It means that the chance of infliction is always 1% less than it should be. It implies that a 50 resistance is actually 49% chance to inflict and that a 0 resistance is a 99% chance to inflict instead of 100%

## Infliction logic differences
The following are all differences with [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) regarding infliction:

- [Freeze](../BattleCondition/Freeze.md) (never used under normal gameplay):
    - If the target has the [Topple](../BattleCondition/Topple.md) condition, it is never removed
    - The target will never have [Drop](../../../Entities/EntityControl/Notable%20methods/Drop.md) called on it even if its [position](../BattlePosition.md) is `Ground` with a `height` above 0.05
    - The infliction is done directly with [SetCondition](SetCondition.md) rather than [AddDelayedCondition](../Delayed%20condition.md#adddelayedcondition)
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called at the end
- [Fire](../BattleCondition/Fire.md) (never used under normal gameplay):
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called at the end
- [Sleep](../BattleCondition/Sleep.md):
    - The target's `sleepres` increase is 8 instead of 9 (10 instead of 13 on [HardMode](../../Damage%20pipeline/HardMode.md))
    - The target will never have [Drop](../../../Entities/EntityControl/Notable%20methods/Drop.md) called on it even if its [position](../BattlePosition.md) is `Ground` with a `height` above 0.05
    - The infliction is done directly with [SetCondition](SetCondition.md) rather than [AddDelayedCondition](../Delayed%20condition.md#adddelayedcondition)
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called at the end
- [Numb](../BattleCondition/Numb.md):
    - On [HardMode](../../Damage%20pipeline/HardMode.md) the target's `numbres` increase is 20 instead of 22. It remains at 17 otherwise
    - The target's `cantmove` is always set to 1, even if its [actimmobile](../Enemy%20features.md#actimmobile) is true
    - The target will never have [Drop](../../../Entities/EntityControl/Notable%20methods/Drop.md) called on it even if its [position](../BattlePosition.md) is `Ground` with a `height` above 0.05
    - The infliction is done directly with [SetCondition](SetCondition.md) rather than [AddDelayedCondition](../Delayed%20condition.md#adddelayedcondition)
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called at the end
- [Poison](../BattleCondition/Poison.md): 
    - There is now a `poisonres` increase by 5 (8 instead when on [HardMode](../../Damage%20pipeline/HardMode.md))
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called at the end
