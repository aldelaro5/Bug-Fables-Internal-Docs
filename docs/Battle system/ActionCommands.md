# ActionCommands
This enum represents all the different action commands implemented in the game through battle actions. The value control the logic of [DoCommand](DoCommand.md).

|Value|Name|Description|
|-----|----|-----------|
|0|None|UNREFERENCED|
|1|[PressKey](Action%20commands/PressKey.md)|An UNUSED and broken simple key press prompt. This command cannot be succeeded.|
|2|HoldKeyBar|UNREFERENCED|
|3|RandomPressKeys|UNREFERENCED|
|4|[TappingKey](Action%20commands/TappingKey.md)|An input mashing prompt where the input can be one of the 4 main ones (Confirm, Cancel, Switch Party or Show HUD) or be a prompt where the player needs to alternate between the left and right input (with caveats, see the sections below for details). Each correct input press increases `barfill` from 0.0 to 1.0. The goal of the command is to make `barfill` reach 1.0 before the `timer` expires.|
|5|[HoldKeyCountdown](Action%20commands/HoldKeyCountdown.md)|A hold input prompt with a number of evenly separated timepoints. The goal of the command is to hold a certain input by the second timepoint and release it before the last timepoint ends. If the input isn't held by the second timepoint or it's released before the last timepoint begins or it's never released before the last timepoint ends, the command results in a failure. This command has forgiveness logic in `demomode` which makes it impossible to fail by guiding the player's input.|
|6|RandomPressKeysTimer|UNREFERENCED|
|7|RandomTappingKeys|UNREFERENCED|
|8|RandomTappingKeysTimer|UNREFERENCED|
|9|[RandomPressBar](Action%20commands/RandomPressBar.md)|A prompt to press the Confirm input timed in such a way that a cursor who goes back and forth on a bar lands in a specific target area. The goal of the command is to hit the input when the cursor falls in that area. The command is failed when the cursor falls outside of the target or if the input isn't pressed before `timer` expires. This command features forgiveness logic in `demomode` where it is impossible to fail the command.|
|10|[RandomTappingBar](Action%20commands/RandomTappingBar.md)|A variation of [TappingKey](TappingKey.md) where the mashing input prompt has a random starting one between Confirm, Cancel or Switch Party (it is not possible to use left/right mode). The input will change to a random one periodically at random intervals. This action command is UNUSED under normal gameplay.|
|11|[RandomPressKeyTimer](Action%20commands/RandomPressKeyTimer.md)|An input prompt of a concealed input that is only revealed after a certain amount of timepoints passes. The goal of the command is to press the input when it is revealed on the last timepoint within 45.0 frames. If any other game inputs (even those outside of the random pool) are pressed during those 45.0 frames or if multiple inputs are pressed at the same time than the correct one or if no inputs is pressed, the command is a failure.|
|12|[MultiPressBar](Action%20commands/MultiPressBar.md)|An UNUSED and stripped down version of [RandomPressBar](RandomPressBar.md) where the cursor only does one sweep of the bar and it has reduced functionality.|
|13|[LongRandomBar](Action%20commands/LongRandomBar.md)|???|
|14|[SequentialKeys](Action%20commands/SequentialKeys.md)|???|
|15|[Crosshairs](Action%20commands/Crosshairs.md)|???|
|16|[PressKeyTimer](Action%20commands/PressKeyTimer.md)|An single input press prompt that lasts a certain amount of frames. The goal of the command is to press the correct input (and only the correct one) on the last 30.0 frames of the command. If no input is taken for the entire duration or if the input is pressed before the last 30.0 frames or if any other inputs recognised by the game is pressed at any point (even if done at the same time than the correct one), the command will be failed. This command has caveats about its UI.|
