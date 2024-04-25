# ActionCommands
This enum represents all the different action commands implemented in the game through battle actions. The value control the logic of [DoCommand](DoCommand.md).

|Value|Name|Description|
|-----|----|-----------|
|0|None|UNREFERENCED|
|1|[PressKey](Action%20commands/PressKey.md)|An UNUSED and broken simple key press prompt. This command cannot be succeeded.|
|2|HoldKeyBar|UNREFERENCED|
|3|RandomPressKeys|UNREFERENCED|
|4|[TappingKey](Action%20commands/TappingKey.md)|An input mashing prompt where the input can be one of the 4 main ones (Confirm, Cancel, Switch Party or Show HUD) or be a prompt where the player needs to alternate between the left and right input (with caveats, see the sections below for details). Each correct input press increases `barfill` from 0.0 to 1.0. The goal of the command is to make `barfill` reach 1.0 before the `timer` expires.|
|5|[HoldKeyCountdown](Action%20commands/HoldKeyCountdown.md)|???|
|6|RandomPressKeysTimer|UNREFERENCED|
|7|RandomTappingKeys|UNREFERENCED|
|8|RandomTappingKeysTimer|UNREFERENCED|
|9|[RandomPressBar](Action%20commands/RandomPressBar.md)|???|
|10|[RandomTappingBar](Action%20commands/RandomTappingBar.md)|A variation of [TappingKey](TappingKey.md) where the mashing input prompt has a random starting one between Confirm, Cancel or Switch Party (it is not possible to use left/right mode). The input will change to a random one periodically at random intervals. This action command is UNUSED under normal gameplay.|
|11|[RandomPressKeyTimer](Action%20commands/RandomPressKeyTimer.md)|???|
|12|[MultiPressBar](Action%20commands/MultiPressBar.md)|???|
|13|[LongRandomBar](Action%20commands/LongRandomBar.md)|???|
|14|[SequentialKeys](Action%20commands/SequentialKeys.md)|???|
|15|[Crosshairs](Action%20commands/Crosshairs.md)|???|
|16|[PressKeyTimer](Action%20commands/PressKeyTimer.md)|???|
