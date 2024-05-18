# `HardCharge`

This action features no action commands or damages logic. It only does the following:

- attacker player party member has its `charge` set to 3

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` set to true
- `Charge7` sound plays
- attacker animstate set to 24 (`Block`)
- ShakeSprite called on attacker player party member with an x intensity of 0.1 for 30.0 of frametimer
- Yield for 0.5 seconds
- The following is performed 3 times:
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on attacker player party member with type 4 (green up arrow)
    - `StatUp` sound plays with a pitch of 0.1 * this loop's index (from 0 to 2) + 0.9 (meaning it does 0.9 first, 1.0 second and 1.1 third)
    - Yield for 0.1 seconds
    - Yield for 0.1 seconds
- attacker player party member has its `charge` set to 3    
