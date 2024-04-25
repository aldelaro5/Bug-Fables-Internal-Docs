# RandomTappingBar
A variation of [TappingKey](TappingKey.md) where ???

> NOTE: This action command will be overriden to [SequentialKeys](SequentialKeys.md) if MainManager.`mashcommandalt` is true (the game settings are configured for Sequential Keys commands instead of the mashing ones).

## `data` array

- `data[0]`: ???

## [DoCommand](../DoCommand.md) Setup phase
The same as [TappingKey](TappingKey.md), but `presskey`'s initial value is random between 4 and 6 inclusive instead of being the value of `data[0]`.

## [DoCommand](../DoCommand.md) Execution phase
The same as [TappingKey](TappingKey.md), but ??? 
