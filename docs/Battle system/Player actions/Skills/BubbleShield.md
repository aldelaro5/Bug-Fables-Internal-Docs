# `BubbleShield`

This action features no action commands or damages logic. It only does the following:

- Inflicts the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition for 1 turn on every player party member whose `hp` is above 0 and who doesn't have an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)
- Increment `playerdata[currentturn].cantmove` (meaning the attacker looses an actor turn)

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` set to true
- Camera moves slowly towards the attacker
- attacker animstate set to 102
- Yield for 0.75 seconds
- attacker animstate set to 119
- Yield for 0.25 seconds
- Camera reset to default
- `Shield` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition for 1 turn on every player party member whose `hp` is above 0 and who doesn't have an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)
- Yield for 0.75 seconds
- `playerdata[currentturn].cantmove` is incremented (meaning the attacker looses an actor turn)
