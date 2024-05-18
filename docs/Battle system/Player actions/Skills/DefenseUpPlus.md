# `DefenseUpPlus`

The same as [DefenseUp](DefenseUp.md), but with the following changes:

- The action is never called from an ItemUsage and requires to be called by skill usage
- The action doesn't just concerns the `target`, it concerns the whole player party. Every player party members whose `hp` is above 0 without an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences) will have AddBuff  called on them which does a [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) to inflict the [DefenseUp](../../Actors%20states/BattleCondition/AttackUp.md) conditiion for 3 turns
- The condition is inflicted for 3 turns instead of 2
- All applicable player party members above will have [StatEffect](../../Visual%20rendering/StatEffect.md) called on them instead of just the `target`
- All applicable player party members will have their y `spin` changed instead of just the `target`
