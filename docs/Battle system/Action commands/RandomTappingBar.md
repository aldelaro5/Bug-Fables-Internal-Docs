# RandomTappingBar
A variation of [TappingKey](TappingKey.md) where the mashing input prompt has a random starting one between Confirm, Cancel or Switch Party (it is not possible to use left/right mode). The input will change to a random one periodically at random intervals. This action command is UNUSED under normal gameplay.

> NOTE: This action command will be overriden to [SequentialKeys](SequentialKeys.md) if MainManager.`mashcommandalt` is true (the game settings are configured for Sequential Keys commands instead of the mashing ones).

> NOTE: This action command exceptionally sets `overridechallengeblock` to true which allows regular blocks to be processed in [DoDamage](../Damage%20pipeline/DoDamage.md) when FRAMEONE is active. This only happens if the action command was not overriden to [SequentialKeys](SequentialKeys.md).

## `timer` usage
Same as [TappingKey](TappingKey.md).

## `data` array

- `data[0]`: This does nothing, but its value must be present or an exception will be thrown
- `data[1]`: Same as [TappingKey](TappingKey.md)
- `data[2]`: Same as [TappingKey](TappingKey.md)
- `data[3]`: Same as [TappingKey](TappingKey.md)

## [DoCommand](../DoCommand.md) Setup phase
The same as [TappingKey](TappingKey.md), but `presskey`'s initial value is random between 4 and 6 inclusive instead of being the value of `data[0]`.

## [DoCommand](../DoCommand.md) Execution phase
The same as [TappingKey](TappingKey.md), but `presskey` now changes to a random value between 4 and 6 inclusive at random interval.

The interval is tracked by a local variable called keytimer which has a starting value being a random number between 70.0 and 100.0 inclusive. It is decreased by the game's frametime each frame of the while loop. When it expires:

- `presskey` changes to a random value between 4 and 6 (the input becomes Confirm, Cancel or Switch Party)
- All the `buttons` have their enablement updated so only the `presskey` one is enabled and the rest disabled
- keytimer is set to be a random number between 70.0 and 150.0 inclusive

Other than this, no other changes are done to the logic.
