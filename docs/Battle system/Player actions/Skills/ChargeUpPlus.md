# `ChargeUpPlus`

The same as [Empower](Empower.md), but with the following changes:

- The action doesn't just concerns the `target`, it concerns the whole player party. Every player party members whose `hp` is above 0 without an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences) will have their `charge` incremented if it was less than 3
- All applicable player party members above will have [StatEffect](../../Visual%20rendering/StatEffect.md) called on them instead of just the `target`
- All applicable player party members will have their y `spin` changed instead of just the `target`
