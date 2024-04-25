# PressKey
An UNUSED and broken simple key press prompt. This command cannot be succeeded.

> NOTE: This action command's logic is broken and is UNUSED under normal gameplay, but the logic remains present

## `data` array

- `data[0]`: The input id to prompt a press for without hold
- `data[1]`: If higher than 0.0, `commandsprites[0]` is disabled. This is optional, `commandsprites[0]` remains enabled if no value exists

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to `data[0]`
- `commandsprites` is set to a new SpriteRenderer array with one element being the SpriteRenderer of a new UI object named `Button` childed to the `GUICamera` with a local position of (-6.0, 1.4, 10.0), scale of (1.3, 1.3, 1.3) using the `guisprites[42]` sprite (white hexagon) with a sortorder of 20
- If `data[1]` exists and is higher than 0.0, `commandsprites[0]` is disabled

## [DoCommand](../DoCommand.md) Execution phase
First, a local inputcooldown variable is initialsed to 0.0.

From there, there the whole logic happens inside a while loop that terminates whenever `killinput` gets set to true which only happens when a result was decided (NOTE: this doesn't do anything in practice because a manual break is done in these cases anyway). This is what happens inside the loop:

- If `caninputcooldown` is above 0.0, `commandsprites[0]`.color is set to pure yellow or to BFBFBF (light gray) otherwise. NOTE: this field isn't initialised by then and it will most likely be 0.0 already
- If the key whose id is `presskey` is pressed (no hold), the input is only accepted if `caninputcooldown` is above 0.0 and the local inputcooldown is 0 or below. If this isn't respected, the input is rejected and instead, the local inputcooldown is set to 20.0. NOTE: since `caninputcooldown` can't get above 0.0 here, this means it is impossible to have the input process, but because it was the only way to succeed the command, it means it's impossible to succeed this command. If it was going to be accepted, the following would have happened:
    - `doingaction` is set to false
    - `commandsuccess` is set to true
    - `caninputcooldown` is set to 0.0
    - 1 second is yielded
    - `killinput` is set to true
    - The logic breaks out of the loop
- Otherwise (`presskey` wasn't pressed without hold), if any keys are pressed (input -4), the command is failed by doing the following:
    - `commandsuccess` is set to false
    - `killinput` is set to true
    - `doingaction` is set to false
    - `caninputcooldown` is set to 0.0
    - 1 second is yielded
    - The logic breaks out of the loop
- If the local inputcooldown is above 0.0, it is decreased by the game's frametime
- `caninputcooldown` is decreased by the game's frametime. NOTE: this happens even if it is 0.0 or below
- A frame is yielded

After the loop is done, `commandsprites[0]`.color is set to BFBFBF (light gray).
