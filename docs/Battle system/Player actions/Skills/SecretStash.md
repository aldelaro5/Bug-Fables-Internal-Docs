# `SecretStash`

This action features no action commands or damages logic. It only does the following:

- [Heal](../../Actors%20states/Heal.md) is called on the `target` player party member with the ammount being 4 + the amount of `HealPlus` [medals](../../../Enums%20and%20IDs/Medal.md) equipped on the `target` player party member
- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the `target` player party member to remove its [Poison](../../Actors%20states/BattleCondition/Poison.md) condition

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` is set to true
- attacker animstate set to 112
- `BagRustle` sound plays
- Yield for 0.5 seconds
- attacker animstate set to 109
- `ItemHold` sound plays
- A new sprite object is created childed to the attacker's `sprite` slightly offset left, up and back without angles. The sprite itself is the `itemsprites` of a random regular [item](../../../Enums%20and%20IDs/Items.md) amongst the following:
    - `CrunchyLeaf`
    - `Mushroom`
    - `AphidEgg`
- Yield for 0.5 seconds
- attacker y `spin` set to 15.0
- Yield for 0.5 seconds
- attacker `spin` zeroed out
- attacker animstate set to 101
- The item sprite object is rooted to the scene
- `Toss` sound plays
- Over the course of 40.0 frames, the item sprite moves the `target` player party member + 2.0 in y via a BeizierCurve3 with a ymax of 7.25
- The item sprite object is destroyed
- The `target` player party member is healed and has its poison removed like mentioned above
- Yield for 0.5 seconds
