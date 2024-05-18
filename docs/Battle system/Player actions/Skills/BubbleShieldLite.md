# `BubbleShieldLite`

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

The same than [BubbleShield](BubbleShield.md), but it has the following changes:

- It assumes a single `target` which will be the only one that will get inflicted the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition for 1 turn if its `hp` is above 0 and it doesn't have an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences). The other player party members will not get the infliction
- `playerdata[currentturn].cantmove` is not incremented meaning the attacker doesn't loose an actor turn
