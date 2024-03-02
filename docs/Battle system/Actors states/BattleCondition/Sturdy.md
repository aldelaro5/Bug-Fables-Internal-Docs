# Sturdy
A condition that prevents other conditions from being inflicted (even through using an [item](../../../Enums%20and%20IDs/Items.md)) while also getting a -3 damage effect applied in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) at the expense that the actor cannot be relayed to.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (`Relay` -> `SelectPlayer`)
This condition will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition prevents the infliction of any other conditions during damage calculation when the target has it except for [Flipped](Flipped.md) if applicable.

This condition also gives a -3 damage effect if the target has it.

## [DoItemEffect](../../../TextAsset%20Data/Items%20data.md#doitemeffect)
This condition prevents the following effects from doing anything:

- `AddPoison`
- `AddNumb`
- `AddFreeze`
- `AddSleep`

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) may be set to 24 (`Block`). Check the method's documentation to learn more.
