# `SharingStash`

The same as [SecretStash](SecretStash.md), but with the following changes:

- The action doesn't just concerns the `target`, it concerns the whole player party. Every player party members whose `hp` is above 0 without an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences) will have the following happen to them:
    - [Heal](../../Actors%20states/Heal.md) is called on the player party member with nosound with the ammount being 2 + the amount of `HealPlus` [medals](../../../Enums%20and%20IDs/Medal.md) equipped on the `target` player party member
    - [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) is called on the player party member to remove its [Poison](../../Actors%20states/BattleCondition/Poison.md) condition
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called on the player party member to inflict the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 2 turns
- There will be multiple item sprites objects initialsed, one for each applicable player party member that will go towards them from the attacker at the same time
- After every applicable player party members were healed, the following happens:
    - `Heal` sound plays
    - Yield for 0.5 seconds
    - `Heal2` sound plays
    - `MagicUp` particles plays on each applicable player party members
    - The final 0.5 seconds yield still happens

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.
