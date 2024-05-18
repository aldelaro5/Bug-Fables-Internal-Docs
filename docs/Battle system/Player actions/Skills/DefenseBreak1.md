# `DefenseBreak1`

This action features no action commands or damages logic. It only does the following:

- AddBuff is called which does a [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) to inflict the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) conditiion on `target` enemy party member for 2 turns

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` set to true
- attacker animstate set to 111
- `FastWoosh` sound plays
- Yield for 0.5 seconds
- AddBuff is called which does a [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) to inflict the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) conditiion on `target` for 2 turns
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `target` with type 3 (blue down arrow)
- A SpriteBounce is added on the `target`'s `rotater` with `speed` of 20.0
- `target` animstate set to 30 (`FakeHurt`)
- `StatDown` sound plays
- `Prefabs/Particles/MagicUp` particles instantiated and played at `target` + 0.5 in y
- Yield for 0.85 seconds
- The SpriteBounce of the `target` is destroyed
- Yield for a frame
- The scale of `target` is reset to its initial value before the action
- Yield for 0.1 seconds
- `Prefabs/Particles/MagicUp` of the `target` moved offscreen and destroyed
- Yield for 0.5 seconds
