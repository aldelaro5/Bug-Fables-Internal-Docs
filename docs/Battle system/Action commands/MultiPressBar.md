# MultiPressBar
An UNUSED and stripped down version of [RandomPressBar](RandomPressBar.md) where the cursor only does one sweep of the bar and it has reduced functionality.

## `timer` usage
This is the amount of frames that the cursor sweep will take. -1.0 is not supported for this command and will result in an immediate failure of the command.

## `data` array

- `data[0]`: The same as [RandomPressBar](RandomPressBar.md)
- `data[1]`: The same as [RandomPressBar](RandomPressBar.md)
- `data[2]`: The same as [RandomPressBar](RandomPressBar.md)
- `data[3]`: The fraction of the game's frametime to decrease `timer` by on each frame

## [DoCommand](../DoCommand.md) Setup phase
The same as [RandomPressBar](RandomPressBar.md).

## [DoCommand](../DoCommand.md) Execution phase
Mostly the same as [RandomPressBar](RandomPressBar.md), but with reduced functionality:

- `timer` isn't doubled at the start of the execution phase
- There is no `Crosshair` sound played
- The cursor only goes from the left of the bar to the right and only once (the command is over when it reaches the rightmost spot)
- There is no `demomode` forgiveness: if the Confirm input is pressed while the cursor is outside the target, it's a failure. Additionally, if Confirm isn't pressed before `timer` expires, it won't restart the command
