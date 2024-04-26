# HoldKeyCountdown
A hold input prompt with a number of evenly separated timepoints. The goal of the command is to hold a certain input by the second timepoint and release it before the last timepoint ends. If the input isn't held by the second timepoint or it's released before the last timepoint begins or it's never released before the last timepoint ends, the command results in a failure. This command has forgiveness logic in `demomode` which makes it impossible to fail by guiding the player's input.

## `timer` usage
This is the amount of time in frames between each timepoints. -1.0 is not a supported value for this action command. This also doubles as the time window the player must be releasing the input on the last timepoint.

## `data` array

- `data[0]`: The amount of timepoints to use (including the last one). NOTE: This must be higher than 1 or a division by 0 exception will be thrown
- `data[1]`: The `presskey` input value the player must be holding and releasing to succeed the command

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to `data[1]`
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-0.8, 1.0, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- `commandsprites` is set to a new array with 1 + `data[0]` elements where the first one is the SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite (small bar container) with a sortorder of 0
- CreatePips(1) is called which initialises each elements of `commandsprites` after the first one (if any exists) to be the SpriteRenderer of a new UI object named `pointX` where `X` is the index of the element childed to `commandsprites[0]` with a local position of (x, 0.0, 0.0) where x is a lerp from 0.0 to 4.5 with a factor being the ratio of element's index - 1 over `commandsprites`.length - 2, scale of (0.5, 0.5, 1.0) (or Vector3.one if it's the last element) using the `guisprites[59]` (white circle) sprite (`guisprites[42]` (white hexagon) if it's the last element) with a sortorder of 1 + the element's index and a color of 7F7F7F (fully opaque gray)

## [DoCommand](../DoCommand.md) Execution phase
This will first yield for 0.15 seconds and then do a loop from 0 to `data[0]` + 1. This means the loop will execute once for each timepoints and an additional one for counting the last one.

The rest is what is contained in the loop.

### Preparations

- `timer` is reset to its initial value when DoCommand was called
- If this is the last loop iteration, `buttons[0]`.`basesprite`.color is set to pure white, otherewise, it's set to 7F7F7F (gray fully opaque)

### `timer` loop
There is an inner while loop that goes on while `timer` hasn't expired yet (decreased by the game's frametime followed by a frame yield on each iteration). The only thing this loop does is idle for all outer loop iterations, except the last one.

On the last outer loop iteration, if by this point, `presskey` wasn't held, the inner loop will break on the next frame by setting `timer` to 0.0 (the next frame yield still goes through, but it ends prematurely after). This has the purpose to break out early in a success condition.

### `demomode` release forgiveness
Normally, if by the time the last iterations of the `timer` loop ends, the player is still holding `presskey`, the command is a fail. However, in `demomode`, there is a forgiveness feature where on the final outer loop iteration, all frames are yielded while `presskey` is held. This will idle infinetely until it is released which will result in success.

### Success condition checks
This is where the command checks if it is done or not and if it is, what result it should be. It proceeds like the following:

- If it's the last iteration and `presskey` is no longer held, this is a success which will do the following:
    - `commandsuccess` is set to true
    - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
    - 0.3 seconds are yielded
- Otherwise, if it's the second iteration or later (meaning between second and before last one) while `presskey` isn't held, this is a failure condition:
    - If on `demomode`, there is a forgiveness where the command will yield all frames until `presskey` is held
    - Otherwise, `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done) followed by an abrupt break of the outer loop

### UI updates
The final step of the loop is that the `commandsprite` whose index is this iteration's index + 1 will have its color changed to pure yellow followed by the `ACBeep` sound being played (or `ACReady` if it's the second to last iteration). This essentially gives the visual feedback of the different timepoints with a different one on the last one.
