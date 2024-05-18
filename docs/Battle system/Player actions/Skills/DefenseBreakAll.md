# `DefenseBreakAll`

The same as [DefenseBreak1](DefenseBreak1.md), but with the following changes:

- The action applies to all enemy party members instead of just the `target` one. This include the AddBuff call and any visual effects or animation logic
- The [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition is inflicted for 3 turns per enemy party members instead of 2 for just the `target` one

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.
