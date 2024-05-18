# `Empower`

This action features no action commands or damages logic. It only does the following:

- If the `target` has less than 3 `charge`, its `charge` is incremented

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` is set to true
- `Magic` sound plays
- `Prefabs/Particles/MagicUp` particles instantiated slightly above the attacker
- attacker animstate set to 4 (`ItemGet`)
- attacker y spin set to -20.0
- `target` player party member's y `spin` set to -15.0
- Yield for 0.75 seconds
- `Prefabs/Particles/MagicUp` particles moved offscreen
- `Prefabs/Particles/MagicUp` particles destroyed in 5.0 seconds
- `StatUp` sound plays at 1.25 pitch
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on the `target` with type 4 (green up arrow)
- If the `target` has less than 3 `charge`, its `charge` is incremented
- `target`'s `spin` is zeroed out
