# `RoyalDecree`

This action features no action commands or damages logic. It only does the following:

- All player party members whose `hp` is above 0 has [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 2 turns with visual effects
- All player party members whose `hp` is above 0 has [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 2 turns with visual effects

## Logic sequence

- Camera moves slowly to look at the player party
- `Prefabs/Objects/ElizantSkill` is instantiated rooted at (-15.0, 0.0, 2.0) and its Animator obtained with a scale of `AntQueen`'s [enddata](../../../TextAsset%20Data/Entity%20data.md#entity-data)'s `startscale`
- Over the course of 71.0 frames, `Prefabs/Objects/ElizantSkill` moves towards -8.0 in x
- The animation clip `1` plays on `Prefabs/Objects/ElizantSkill`'s Animator
- Yield for 0.5 seconds
- All player party members whose `hp` is above 0 has [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 2 turns with visual effects
- Yield for 0.5 seconds
- All player party members whose `hp` is above 0 has [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 2 turns with visual effects
- Yield for 0.5 seconds
- `Prefabs/Objects/ElizantSkill` MoveTowards -15.0 in x over the course of 50.0 frames with smooth without local (MainManager.MoveTowards)
- `Prefabs/Objects/ElizantSkill` is destroyed in 1.0 seconds
