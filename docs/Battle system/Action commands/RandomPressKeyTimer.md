# RandomPressKeyTimer
An input prompt of a concealed input that is only revealed after a certain amount of timepoints passes. The goal of the command is to press the input when it is revealed on the last timepoint within 45.0 frames. If any other game inputs (even those outside of the random pool) are pressed during those 45.0 frames or if multiple inputs are pressed at the same time than the correct one or if no inputs is pressed, the command is a failure.

## `timer` usage
This is the amount of frames between the timepoints with the exception of the last one which always takes 45.0 frames. -1.0 is not supported and will result in a failure of the command.

## `data` array

- `data[0]`: Decides the sets of inputs in the random pool:
    - 0.0: Random between 4 and 6 inclusive (Confirm, Cancel and Switch party)
    - 1.0: Random between 0 and 3 inclusive (only the 4 directional inputs)
    - 2.0: Random between 0 and 6 inclusive
- `data[1]`: The amount of timepoints to use including the last one. NOTE: This must be higher than 1 or an exception will be thrown

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to a value that depends on `data[0]` (check the `data` array section above for details)
- `commandsprites` is set to a new array with 2 + `data[1]` elements where the first one is the SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite (UI bar container) with a sortorder of 0
- CreatePips(2) is called which initialises each elements of `commandsprites` after the second one (if any exists) to be the SpriteRenderer of a new UI object named `pointX` where `X` is the index of the element childed to `commandsprites[0]` with a local position of (x, 0.0, 0.0) where x is a lerp from 0.0 to 4.5 with the factor being the ratio of element's index - 2 over `commandsprites`.length - 3, scale of (0.5, 0.5, 1.0) (or Vector3.one if it's the last element) using the `guisprites[59]` sprite (a white cirlce) (or `guisprites[42]` (white hexagon) if it's the last element) with a sortorder of 1 + the element's index and a color of 7F7F7F (fully opaque gray)
- letters is set to a new array of one element childed to the last `commandsprites` which has the following initialisation:
    - SetFont is called on it with [fontid](../../SetText/Notable%20states.md#font-id-table) 0 (`BubblegumSans`)
    - text of `?`
    - local position of Vector3.zero
    - scale of (0.1, 0.1, 1.0)
    - layer of 5 (`UI`)
    - anchor of `MiddleCenter`
    - MeshRenderer has a sortingOrder of 30
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` (disabled for now) which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (0.0, 0.0, 0.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 50
    - parentobj: The last `commandsprites`

## [DoCommand](../DoCommand.md) Execution phase
This begins by yielding for 0.15 seconds.

From there, the entire execution is contained in a loop that runs for `data[1]` + 1 times. The loop is only exited when an outcome has been determined.

### Before inner `timer` loop

- `timer` gets set to initialtimer
- If this is the last loop iteration, the input is revealed by doing the following:
    - The `ACReady` sound is played
    - `timer` gets set to 45.0
    - `buttons[0]` gets enabled (this reveals the input to press)
    - `letters[0]` (the `?`) gets destroyed if it's still present

### Inner `timer` loop
Most of the core logic of the command occurs between 2 timepoints which is tracked by this inner loop. It goes on until `timer` expires.

What happens here depends if it's the last outer iteration or not. If it's not, `timer` gets decreased by the game's frametime and a frame is yielded (this is basically just an idle loop).

If it's the last outer iteration however, 3 possible things can happen (mutually exclusive, the first one happens per inner loop iteration):

- If `presskey` is pressed without hold and it is the only one pressed without hold, the commmand is a success:
    - `commandsuccess` is set to true
    - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
    - The command breaks out of the `timer` loop
- Otherwise, if any of the game's recognised inputs is pressed except `presskey`, the command is a failure:
    - `commandsuccess` is set to false
    - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
    - The command breaks out of the `timer` loop
- If neither of the above applies, this is just an idle iteration where `timer` is decreased by the game's frametime followed by a frame yield

### After `timer` loop
The corresponding `commandsprites` of the current timepoint has its color set to pure yellow and if it's not the last iteration, the `ACBeep` sound is played.
