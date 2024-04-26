# RandomPressBar
A prompt to press the Confirm input timed in such a way that a cursor who goes back and forth on a bar lands in a specific target area. The goal of the command is to hit the input when the cursor falls in that area. The command is failed when the cursor falls outside of the target or if the input isn't pressed before `timer` expires. This command features forgiveness logic in `demomode` where it is impossible to fail the command.

## `timer` usage
This is the amount of frames allowed before considering the commmand a failure (not applicable in `demomode`) when `data[3]` is 2.0 (being higher or lower means that a "frame" takes more or less time). -1.0 is not supported for this command and will result in an immediate failure of the command.

## `data` array

- `data[0]`: The lower bound of the starting bar target area in percentage (from 0.0 to 100.0)
- `data[1]`: The upper bound of the starting bar target area in percentage (from 0.0 to 100.0)
- `data[2]`: The length of the bar target area in percentage (from 0.0 to 100.0)
- `data[3]`: The fraction of the game's frametime to decrease `timer` by on each frame. It also serve as a multiplier to how fast the cursor moves back and forth. An expected value for this is 2.0 because `timer` gets doubled which would compensate for this.

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4 (Confirm)
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-0.8, 1.0, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- internaldata is set to a new array of 2 elements:
    - 0: A random float between `data[0]` and `data[1]`
    - 1: 0.0
- `commandsprites` is set to a new array with 4 elements:
    - 0: The SpriteRenderer of a new UI object named `barbottom` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of (5.0, 1.0, 1.0) using the `guisprites[58]` sprite (a white small bar) with a sortorder of 0 and a color of clear (all transparent black)
    - 1: The SpriteRenderer of a new UI object named `barmiddle` childed to the `commandsprites[0]` with a local position of (`internaldata[0]` / 100.0, 0.0, 0.0), scale of (`data[2]` / 100.0, 1.0, 1.0) using the `guisprites[58]` sprite (a white small bar) with a sortorder of 1 and a color of 60E275 (mostly clear green)
    - 2: The SpriteRenderer of a new UI object named `cursor` childed to the `commandsprites[0]` with a local position of (0.0, 0.25, 0.0), scale of (0.45, 0.1, 1.0) using the `cursorsprite[0]` sprite (the leaf cursor sprite) with a sortorder of 2 and with angles of (0.0, 0.0, 270.0) which rotates it such that it is pointing down
    - 3: The SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite (a small UI bar container) with a sortorder of 0

## [DoCommand](../DoCommand.md) Execution phase
This execution mostly occurs in a while loop with logic before and after.

### Before `timer` loop

- 0.35 seconds are yielded
- The `Crosshair` sound is played on `sounds[9]` with 0.9 pitch and 0.25 volume on loop
- `timer` is doubled (this compensate for the expected value of 2.0 of `data[3]`)
- A temptimer local float is set to 0.0 which tracks where the cursor should fall into in relation to initialtimer

### `timer` loop
This is a while loop that executes as long as `timer` hasn't expired yet:

- `commandsprites[2]` (the cursor) local x position gets set to temptimer / initialtimer
- If `presskey` is pressed without hold, there are 3 possible scenarios (mutually exclusive, the first one applies):
    - If `commandsprites[2]` (the cursor) local x position * 100.0 is between internaldata\[0\] and internaldata\[0\] + `data[2]` (meaning it's in the target area), the command succeeds by doing the following:
        - `commandsuccess` is set to true
        - `buttons[0]`.`basesprite`.color is set to 7F7F7F (gray fully opaque)
        - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
        - `sounds[9]` (the `Crosshair` sound) is stopped
        - A second is yielded
        - Break out of the loop
    - Otherwise, if we are in `demomode`, PlayBuzzer is called. This is the forgiveness feature because this would have normally failed the command
    - Otherwise (the cursor is outside of the target while we aren't in `demomode`), the command is failed by doing the following:
        - `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
        - `sounds[9]` (the `Crosshair` sound) is stopped
        - A second is yielded
        - Break out of the loop
- `timer` is decreased by a fraction of the game's frametime contained in `data[3]` (this is normally 2.0 which compensates from the fact its value was doubled earlier)
- temptimer is incremented by a fraction of the game's frametime. This fraction is `data[3]`, but if `timer` / initialtimer is 1.0 or lower (meaning the cursor is on the way back), the fraction is -1 * `data[3]` instead so it will decrease instead of increasing
- If we are in `demomode` while the `timer` expired, `timer` is reset to initialtimer * 2.0 and temptimer is reset to 0.0. This essentially restarts the command as if nothing happened without failing
- A frame is yielded

### After `timer` loop

- `sounds[9]` (the `Crosshair` sound) is stopped
