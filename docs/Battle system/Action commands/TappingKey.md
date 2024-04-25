# TappingKey
An input mashing prompt where the input can be one of the 4 main ones (Confirm, Cancel, Switch Party or Show HUD) or be a prompt where the player needs to alternate between the left and right input (with caveats, see the sections below for details). Each correct input press increases `barfill` from 0.0 to 1.0. The goal of the command is to make `barfill` reach 1.0 before the `timer` expires.

> NOTE: This action command will be overriden to [SequentialKeys](SequentialKeys.md) if MainManager.`mashcommandalt` is true (the game settings are configured for Sequential Keys commands instead of the mashing ones).

> NOTE: This action command exceptionally sets `overridechallengeblock` to true which allows regular blocks to be processed in [DoDamage](../Damage%20pipeline/DoDamage.md) when FRAMEONE is active. This only happens if the action command was not overriden to [SequentialKeys](SequentialKeys.md).

## `timer` usage
This represents the amount of frames to give to the player before the command fails. A value of -1 means inifnite time where it is not possible to fail the command.

## `data` array

- `data[0]`: If after being truncated to int, it's -1, this is a left/right input mashing prompt. If it's between 0 and 3 inclusive after truncating to int, it's an input mashing prompt whose id corresponds to the value + 4. NOTE: any value of 4.0 or above is considered invalid and will cause an exception to be thrown. This is because only the 4 face buttons inputs (confirm, cancel, switch and show HUD) are supported which have ids from 4 to 7
- `data[1]`: A multiplier that controls how much `barfill` is incremented each correct input press. It changes the fraction of the game's frametime to use in the calculation formula meaning 1.0 will use the game's frametime directly. Additionally, if `data[0]` is not -1 after being truncated to int, `data[2]` will effectively be clamped from 0.005 to 0.0025 * `data[1]` when calculating the decrease of `barfill` every frame. The specific increase formula of `barfill` on a correct input press depends on the game's settings:
    - With VSync off and the framerate set to 60 FPS, 1.0 means `barfill` increases by the game's frametime / 80.0
    - With VSync off and the framerate set to 30 FPS, 1.0 means `barfill` increases by the game's frametime / 40.0
    - With Vsync on, 1.0 means `barfill` increases by the game's frametime / 80.0 * (The monitor's refresh rate / 60.0). NOTE: This implies that the monitor's refresh rate acts as a multiplier on top of `data[1]`'s value if VSync is on
- `data[2]`: If `data[0]` is not -1 after being truncated to int, `barfill` will decrease every frame by a fraction of the game's frametime equal to 0.005 * this value. However, when calculating the decrease, it will be clamped from 0.005 to 0.0025 * `data[1]`. If `data[0]` is -1 after being truncated to int, this does nothing
- `data[3]`: This value does nothing, but its existence in the array is mandatory because if it is not present, it becomes impossible for the command to succeed, even if `barfill` reaches 1.0

## High level explanation
Since the logic of this action command is complex, a high level explanation is provided. For the details, check the setup and execution phase's details.

There are 2 ways to operate this action command: either `data[0]` after being truncated to int is -1 or it isn't (but still lands between 0 and 3 inclusive). 

If it is -1, it's a left/right mashing prompt where `barfill` only increments when left and right are pressed one after the other. The rate of increment is controlled by `data[1]` (typical values used under normal gameplay are between 6.0 and 8.0). `data[2]` is unused in this mode, but still must exist because `data[3]` needs to exist in order for the command to function (otherwise, it's impossible to succeed the command).

If it's between 0 and 3 inclusive, it's a single input mashing prompt where the input whose id is 4 + `data[0]` needs to be pressed in order for `barfill` to increase (same usage of `data[1]` here than in the left/right mashing prompt). However, this mode also makes `barfill` decrease at a rate controlled by `data[2]`, but that rate cannot exceed half of `data[1]`. Typical values are similar than `data[1]`'s values.

Each mode features 2 `commandsprites`: one for the bar's container and one for the bar itself. The bar's x scaling will get adjusted depending on `barfill` which is itself clamped from 0.0 to 1.0 (0.0 being empty and 1.0 being fully filled). Additionally, as an additional audio feedback, as `barfill` gets higher, so does the pitch of a looping `Barfill` sound that will go from 1.0 to 2.0.

To succeed the command (assuming `data[3]` exists), `barfill` must reach 1.0 before the timer sent expires or the command is failed. There is no timer if it is -1.0 meaning the command cannot be failed in such a situation.

### Left/right mashing prompt
This mode features:

- `presskey` starting at 2 (left) and its value swaps between 2 (left) and 3 (right) whenever an input is accepted
- 2 `buttons` which shows the left and right glyphs. They will periodically alternate between being white and gray every 8 frames, but this is only for visual indication purposes: it doesn't affect the logic in any way

The way this works is the starting value of `presskey` is 2 (left) and it is the input that will always be accepted before moving on to the next one. In an ideal scenario, the player starts with left which increases `barfill` and causes `presskey` to become 3 (right) which the player can press and make `presskey` become 2 (left) again. This continues until the bar is filled or timer expires.

However, the game allows for 1 input of tolerance as a grace input. It means for only a single time in a row, even if `presskey` is 2 (left), the game will accept up to 1 right input and even count it towards increasing `barfill`. The same happens for the reverse: if `presskey` is 3 (right), up to 1 left input is allowed. This can only be granted once in a row: once a grace input is taken, the game will no longer accept any inputs except the `presskey` one. This allows the player to completely skip one input, but still get back on track on the next one. It incidentally allows to double tap each input because the incorrect one is allowed each time.

There are some caveats with this mashing prompt however:

- Due to the tolerance logic on keyboard, it is possible to press both left and right input at the same time every time and still have it registered because the game will prioritise the correct input, but will still allow the incorrect one once before the next correct input press
- Due to an issue with MainManager's call to InputIO on controllers, if the player manages to swap the left/rights input to be pressed/released within a frame (such as flicking a D-pad), the game will not register the new inputs even if the correct one is being pressed. This is because the mash prompt only accept non holding inputs and MainManager does not distinguish on controllers (not keyboards) between left or right being held. It only signals to the game that either is held or neither are which isn't sufficient for this. This can lead to frequent inputs drops.

### Single input mashing prompt

This mode features:

- `presskey` starting stays at 4 + `data[0]` which is the input the prompt will be about and the only one that will be accepted to increase `barfill`
- 4 `buttons`, each corresponding to the Confirm, Cancel, Switch and Show HUD inputs glyphs. However, only the `presskey` one is enabled, the rest are disabled and thus, not shown. Each 3 frames, the `buttons` wll have their colors changed from gray to white and then back to gray, but this is only for visual indication purposes: it doesn't affect the logic in any way

This is a simpler mode: only the `presskey` input is accepted, each increases `barfill`, but on every frames (even those where `presskey` isn't pressed), `barfill` gets decreased by a slower amount. The player needs to mash fast enough to fight the decrease and fill the bar.

The only requirement for the input to be accepted is that it wasn't held meaning it is a fresh new press. It is however possible to bypass this restriction by alternating between a controller and a keyboard because their input holding are tracked separately.

## [DoCommand](../DoCommand.md) Setup phase

- The `Barfill` sound is played on `sounds[4]` at 0.7 volume on loop
- `presskey` is set to `data[0]` (or to 2 if it was -1 meaning the left input is the starting one in left/right input prompt)
- `buttons` is set to a new array of ButtonSrpites (length 2 if `data[0]` is -1, length 4 if it's not). Each are initialised to be a new GameObject named `ButtonX` where `X` is the index of the element. The GameObject is enabled if `data[0]` is -1 or if `data[0]` - 4 is the element's index (disabled otherwise). This means that if it's a left/right input prompt, both `buttons` are enabled, but if it's a face button input prompt, only the concerned one is left enabled with the others left disabled. Each GameObject will have a ButtonSprite attached with SetUp called with the following:
    - buttonid: if `data[0]` is -1, it's 2 + the element's index (meaning 2 or 3 which are the left/right inputs) and if it's not -1, it's 4 + the element's index (meaning from 4 to 7 which are the face buttons inputs)
    - onlytype: -1
    - description: empty
    - position: (2.35, 1.5, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- `commandsrpites` is set to a new array of 2 SpriteRenderer:
    - `commandsprites[0]`: The SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.5, 10.0), scale of Vector3.one using the `guisprites[81]` sprite (an action command bar container) with a sortorder of 0
    - `commandsprites[1]`: The SpriteRenderer of a new UI object named `bar` childed to `commandsprites[0]` with a local position of Vector3.zero, scale of (0.0, 1.0, 1.0) using the `guisprites[82]` sprite (a white bar) with a sortorder of 1 and a color of pure yellow

## [DoCommand](../DoCommand.md) Execution phase
There's 3 parts: before the while loop, the while loop itself and after it.

### Pre while loop

- `overridechallengeblock` is set to true
- A local tolerance int is initialised to 1
- A local altbutton int is initialised to 3

### While loop
This loop is the main execution of the command and it runs while timer is above 0.0 meaning there's still time left (or we are operating in infinite mode where this goes on until `barfill` is full).

This is what happens inside the loop:

- `sounds[4]`.pitch is set to `barfill` + 1.0 clamped from 1.0 to 2.0 (meaning the pitch / speed goes higher and faster as the bar gets filled until doubled pitch / speed)
- `commandsprites[1]` x scale is set to a lerp from the existing one to `barfill` with a factor of 1/5 of the game's frametime (this visually aligns the bar to the `barfill` value smoothly)
- If the timer floored is divisible by 3 (or by 8 if `data[0]` is -1) and internaldata\[0\] is 0.0 or below:
    - The `buttons`'s `basesprite`.color is changed to pure white or 7F7F7F (gray) depending on conditions:
        - If `data[0]` is -1, `buttons[0]` and `buttons[1]` have each their color set to white or gray if the first one's `basesprite` isn't null. If the first one was gray, it becomes white with the other becoming gray and vice versa otherwise. Additionally, `buttons[1]` local position is set to (3.5, 1.5, 10.0)
        - Otherwise, if `buttons[presskey - 4]`'s `basesprite` isn't null, its color gets set to white if it was grey or set to grey otherwise
    - `internaldata[0]` is set to 3.0
- If `presskey` is pressed without hold OR (the altbutton is while `data[0]` is -1 and tolerance is above 0):
    - `barfill` is incremented by a fraction of the game's frametime. This fraction is `data[1]` / 80.0 * the framerate (or the refresh rate if MainManager.`vsync` isn't 0) / 60.0. NOTE: This vsync logic is incorrect: it essentially means the monitor's refresh rate acts as a speed multiplier where the higher it is and away from 60.0, the faster each press will fill the bar
    - If `data[0]` is -1, then it depends if `presskey` was pressed or not. If it wasn't, tolerance is decremented. If it was, tolerance is set to 1 and both `presskey` and altbutton are set to flip flop between 2 or 3 (so if it was 2, it becomes 3 and vice versa)
- If `data[0]` isn't -1, `barfill` is decremented by value that is a half of a fraction of the game's frametime. This fraction is `data[2]` clamped from 1.0 to `data[1]` / 2.0 and then the value is divided by 100.0
- `barfill` is clamped from 0.0 to 1.0
- If `barfill` is at least 1.0 and `data` has more than 3 elements, the command succeed with the following:
    - The `ACReady` sound is played
    - `commandsuccess` is set to true
    - `commandsprite[1]`.color is set to pure green
    - `commandsprite[1]` scale is set to Vector3.one
    - The logic exits out of the loop
- internaldata\[0\] is decreased by the game's frametime
- timer is decreased by the game's frametime
- A frame is yielded

### Post while loop

- `sounds[4]` is faded out by 1/10 of the game's frametime every frame
- All `buttons` elements are disabled
- 0.35 seconds are yielded
