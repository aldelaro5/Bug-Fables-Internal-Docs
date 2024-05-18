# `Cleanse`

This action features no action commands or damages logic. It only does the following:

- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on `target` enemy party member

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` set to true
- attacker animstate set to 111
- `FastWoosh` sound plays
- Yield for 0.5 seconds
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on `target` enemy party member
- DeathSmoke particles play on the `target` enemy party member with uniform 3x scale
- A SpriteBounce is added on the `target`'s `rotater` with `speed` of 20.0
- `target` animstate set to 30 (`FakeHurt`)
- `Heal3` sound plays
- `Prefabs/Particles/MagicUp` particles instantiated and played at `target` + 0.5 in y
- Yield for 0.85 seconds
- The SpriteBounce of the `target` is destroyed
- Yield for a frame
- The scale of `target` is reset to its initial value before the action
- Yield for 0.1 seconds
- `Prefabs/Particles/MagicUp` of the `target` moved offscreen and destroyed
- Yield for 0.5 seconds
