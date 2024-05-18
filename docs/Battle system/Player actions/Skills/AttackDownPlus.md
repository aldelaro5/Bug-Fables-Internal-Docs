# `AttackDownPlus`

The same as [AttackDown](AttackDown.md), but with the following changes:

- The action applies to all enemy party members instead of just the `target` one. This include the AddBuff call and any visual effects or animation logic
- The [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition is inflicted for 3 turns per enemy party members instead of 2 for just the `target` one

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.
