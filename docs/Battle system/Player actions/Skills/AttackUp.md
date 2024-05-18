# `AttackUp`

This action features no action commands or damages logic. It only does the following:

- AddBuff is called which does a [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) conditiion on `target` for 2 turns

## [Battle](../../Battle%20flow/Action%20coroutines/UseItem.md#battle) ItemUsage
This action will be processed if a `VitalitySeed` [item](../../../Enums%20and%20IDs/Items.md) is used with a `Battle` ItemUsage. If this is what caused this action, it features less animation logic.

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` is set to true
- If [currentaction](../../Player%20UI/Pick.md) is `ItemList` (the action was called from item usage):
    - Yield for 0.25 seconds
- Otherwise (the action was called from skill usage):
    - `Magic` sound plays
    - `Prefabs/Particles/MagicUp` particles instantiated slightly above the attacker
    - attacker animstate set to 4 (`ItemGet`)
    - attacker y spin set to -20.0
    - `target` player party member's y `spin` set to -15.0
    - Yield for 0.75 seconds
    - `Prefabs/Particles/MagicUp` particles moved offscreen
    - `Prefabs/Particles/MagicUp` particles destroyed in 5.0 seconds
- `StatUp` sound plays
- AddBuff is called which does a [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) conditiion on `target` for 2 turns
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on the `target` with type 0 (red up arrow)
- `target`'s `spin` is zeroed out
