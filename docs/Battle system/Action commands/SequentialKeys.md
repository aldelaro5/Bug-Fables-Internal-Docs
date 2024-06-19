# SequentialKeys
A series of simple random input prompts that have to be pressed in sequence in a row. The goal of the command is to press all inputs in the sequence correctly in a row under a certain amount of time. If too much time passes without completing all inputs or if the wrong input is pressed, the command is a failure. This command supports 2 optional features: the ability to make it go on infinetely until success and a feature that hides future inputs until all previous ones are entered correctly.

## `infinitecommand` feature
This is the only action command to support the `infinitecommand` feature. [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) resets this field to false when called, but an action can set it to true to opt in to the infinite feature. This allows the command to never end until it is succeeded meaning it will restart if it is failed. NOTE: This feature doesn't work correctly if the command is configured to hide future inputs because it will not reset their visibility if the command is failed.

## `timer` usage
The amount of frames of the total time allowed to enter all the inputs before the command is failed. -1.0 is not supported and it will result in the immediate failure of the command (or a softlock if `infinitecommand` is true).

## `data` array

- `data[0]`: The amount of inputs that must be pressed correctly in a row for the command to be succeeded
- `data[1]`: This value does nothing, but its existance allows to opt-in to a feature where all the inputs except the first one are hidden and the next one will get revealed as correct inputs are entered. This feature is optional: to opt out of it, `data[1]` must not exist

## Odd results output
This command changes 3 results fields, but 2 of them are unexpected:

- `commandsuccess`: Reports success or failure as usual, nothing unexpected
- `combo`: Reports the amount of inputs correctly pressed in a row. Resets to 0 each failure of an `infinitecommand`. While this field is normally used for [ShowSuccessWord](../Visual%20rendering/ShowSuccessWord.md), its value will get written to by this command
- `barfill`: Reports 1.0 on success. On failure, it reports an unexpected value being a value lerped from 0.0 to 1.0 with a factor being the last input index correctly pressed / 6.0

## [DoCommand](../DoCommand.md) Setup phase

- `overridechallengeblock` is set to true
- The `Barfill` sound is played on `sounds[9]` with a volume of 0.5 on loop
- `commandsprites` is set to a new array with 3 + ((int)`data[0]` - 1) elements (or just 3 if `data` has only 1 element), here are the first 3 elements:
    - 0: The SpriteRenderer of a new UI object named `barbottom` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite (small UI bar container) with a sortorder of 0
    - 1: The SpriteRenderer of a new UI object named `bar` childed to the `commandsprites[2]` (it was created before this one) with a local position of Vector3.zero, scale of Vector3.one using the `guisprites[58]` sprite (white bar) with a sortorder of 10 and a color of pure yellow
    - 2: The SpriteRenderer of a new UI object named `barbottomsmall` childed to the `commandsprites[0]` with a local position of (2.0, 1.0, -0.1), scale of (2.0, 1.0, 1.0) using the `guisprites[64]` sprite (small red bar) with a sortorder of 2
- intdata is set to a new array with length being `data[0]` where each elements is set to be a random number between 0 and 6 inclusive
- `buttons` is set to a new array with length `data[0]` where each is initialised to the ButtonSprite of a new GameObject named `ButtonX` where `X` is the index, `shrunkkey` set to true if the matching intdata is a LongButton (left unchanged otherwise) and it has SetUp called on it with the following:
    - buttonid: The matching intdata element
    - onlytype: -1
    - description: empty
    - position: (x, y, 0.8) where:
        - x is a lerp between 0.0 - offset to 5.0 + offset with a factor of 1.0 / (`data[0]` - 1.0) * index. offset is 0.0 unless one of the intdata is a LongButton where it's 0.45
        - y is 0.0 unless it is the first `buttons` or `data` has exactly 1 element where the value is 999.0 instead
    - iconsise: (0.65, 0.65, 0.65)
    - sortorder: 2
    - parentobj: `commandsprites[0]`
- If `data` has more than 1 element,  every `commandsprites` with an index of 3 or higher are initialised to the SpriteRenderer of a new UI object named `q` childed to the `commandsprites[0]` with a local position of (x, 0.0, 8.0) where x is a lerp from 0.0 to 5.0 with a factor of 1.0 / (`data[0]` - 1.0) * (index - 2), scale of (0.75, 0.75, 0.75) using the `guisprites[171]` sprite (a question mark sprite) with a sortorder of 10
- `presskey` is set to `intdata[0]`

## [DoCommand](../DoCommand.md) Execution phase
The entire execution phase is enclosed in a while(true) loop with the exception that after the loop is done, `sounds[9]` (`Barfill` sound) is stopped.

There are 3 parts inside this loop: setup, `timer` loop and reset.

### Setup

- The index of the current input index is tracked by a local int starting at 0
- `combo` is set to 0
- A frame is yielded
- All `buttons` except the first one has their `basesprite`.color set to gray

### `timer` loop

- `sounds[9]` (`Barfill` sound)'s pitch is set to 0.5 + (1.0 - `timer` / initialtimer) clamped from 1.0 to 1.5 (the pitch gets higher as `timer` goes down)
- `commandsprites[1]` (the decreasing bar)'s x scale is set to `timer` / initialtime
- If the current input (tracked in `intdata`) is pressed without hold, the current input is advanced by doing the following:
    - The corresponding `buttons` of the current input's `basecolor`.color is set to pure green
    - If `data[1]` exists (future inputs were hidden) and we aren't on the last input:
        - The next `buttons`'s input's local position is set to the corresponding `commandsprites` one (which has a question mark sprite)
        - The corresponding `commandsprites` (question mark sprite) is disabled
    - The current input index is incremented
    - `combo` is incremented
    - `barfill` is set to a lerp from 0.0 to 1.0 with a factor of the current input index / 6.0
    - A sound is played depending on the new input index:
        - If it's not the last one, the `ACBeep` sound is played at a pitch of 1.0 + 0.1 * current input index (the pitch gets higher the later the input is)
        - If it's the last one, the `ACReady` sound is played at 1.0 pitch
    - If the new input index isn't the last one, the corresponding `buttons` of the current input's `basecolor`.color is set to pure white
- Otherwise, if any inputs the game recognises is pressed without hold, the command is failed by doing the following:
    - The corresponding `buttons` of the current input's `basecolor`.color is set to pure red
    - `commandsuccess` is set to false
    - PlayBuzzer is called
    - If it's not an infinite command:
        - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
        - `sounds[9]` (`Barfill` sound) is stopped
    - 0.5 seconds are yielded
    - We break out of the `timer` loop (this will break out of the outer loop too if it's not an `infinitecommand`)
- If we are past the last input in intdata, the command is succeeded by doing the following:
    - `commandsuccess` is set to true
    - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
    - `inifinitecommand` is reset to false
    - `barfill` is set to 1.0
    - `sounds[9]` (`Barfill` sound) is stopped
    - 0.5 seconds are yielded
    - We break out of the `timer` loop (this will also break out of the outer loop because of `commandsuccess` being true)
- `timer` is decreased by the game's frametime
- A frame is yielded

### Reset
If we reach this point, it means an outcome was determined and if it is a success by `commandsuccess` being true, then we break out of the loop and the action command is done.

If it was a fail, then it depends on `infinitecommand`: if it's false, then we break out of the loop early, but if it's true, a new iteration of the loop is prepared by doing the following because the command will continue infinetely as long as it isn't succeeded:

- `timer` is reset to initialtimer
- `barfill` is reset to 0.0
- `buttons[0].basesprite`.color is set reset to pure white
