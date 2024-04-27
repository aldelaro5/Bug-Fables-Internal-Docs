# LongRandomBar
A multi press input prompt spread over multiple timepoints as a cursor sweeps once through a bar containing the timepoints which are randomly generated. The goal of the command is to press Confirm while the cursor is close enough to the timepoint and hit as many as possible. A timepoint is not hit if no inputs was pressed or if the input was pressed while the cursor was too far off the timepoint. This command cannot be failed: it doesn't use `commandsuccess` for the outcome, but rather `successfullchain` which will report the amount of timepoints that was hit sucessfully.

## `timer` usage
The percentage of the full bar to advance the cursor on each frame. This must be higher than 0.0 and lower than 1.0 for the command to work correctly (-1.0 is not supported). A typical value the game uses for this is 0.01 meaning the bar sweep will last for 100.0 frames.

## `data` array

- `data[0]`: The amount of timepoints to have on the bar after truncation to int. NOTE: This needs to be at least 1 for the command to work and it must not exceed 4 or it will become possible that 2 timepoints overlap. See the section below for details. Also, never put a decimal portion of the value of .5 or higher because it will lead to an exception being thrown
- `data[1]`: Determines the first timepoint's minimum position. NOTE: This must be between 0.0 and 2.0625, otherwise, it's possible that the first timepoint overlap with another. See the section below for details

## Known range math issues
The math done to calculate the range the timepoints will fall into is broken. It only works cumutively from left to right and the only guarantee the logic gives is that the first timepoint's max range is 2.0625 and the last timepoint's max range is 8.25 (the bar's entire range is from 0.0 to 8.55).

Anything else is determined either by config or by harcoding based on the previous range:

- The first's min range is the value of `data[1]` which means it must not exceed 2.0625 in order to not have an overlap
- Each timepoints after the first one has a min range equal to the previous timepoint + 1.0
- Each timepoint from the second one to the second to last one has a max range of 8.25 / (`data[0]` - timepoint index)

That second and third clauses creates problems when there are 5 or more timepoints because it now becomes possible that 2 timepoints overlap due to incorrect range math. At 4, it's still wrong, but by coincidence, the worst case only involves the second point having a reduced possible range, but it is guaranteed to not overlap. Overlapping is not handled well by the logic and will look visually erreneous.

Therefore, it leads to the following recommendations to avoid overlaps:

- `data[0]` must be lower than 4, but it also needs to be higher than 1 for relatively distributed ranges, and higher than 0 for the command to function in general (1 will have a single point at the start of the bar, but nor further than ~25% in).
- `data[1]` must be between 0.0 and 2.0625

## Known hexagon UI issue
Due to the game using hexagon shaped indicator for the timepoint, it becomes visually ambiguous the exact range of each timepoints. Visually, it was found with empirical testing that the range falls within the vertical rectangle of the hexagon and must not fall into the 2 small triangles on the left and right. The cursor should be touching any of the 2 shades of grey.

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4
- `successfulchain` is set to 0
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-5.2, 2.35, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- `commandsprites` is set to a new array with 3 + `data[0]` elements, here are the first 3 elements:
    - 0: The SpriteRenderer of a new UI object named `barbottom` childed to the `GUICamera` with a local position of (-4.45, 2.4, 10.0), scale of Vector3.one using the `guisprites[81]` sprite (long bar UI container) with a sortorder of 0
    - 1: The SpriteRenderer of a new UI object named `barslide` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (0.0, 1.0, 1.0) using the `guisprites[82]` sprite (white bar) with a sortorder of 1 and a color of pure yellow
    - 2: The SpriteRenderer of a new UI object named `cursor` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (0.5, 0.5, 0.5) using the `cursorsprite[0]` sprite (the leaf UI cursor) with a sortorder of 20 and with angles of (180.0, 0.0, 90.0) (it points down)
- internaldata is set to a new float array with length of `data[0]` (this local will contain the timepoints)
- For each internaldata elements:
    - The element is initialised to a random number between the previous one + 1.0 and 8.25 / `data[0]` - the index (if it's the first element, it's random between `data[1]` and 2.0625)
    - `commandsprites[3 + X]` (where `X` is the index) is initialised to be the SpriteRenderer of a new UI object named `Button` childed to the `commandsprites[0]` with a local position of (`internaldata[Y]`, 0.0, 0.1) where Y is the index, scale of (0.65, 0.65, 0.65) using the `guisprites[42]` sprite (white hexagon) with a sortorder of 2 + index and with a color of 7F7F7F (gray fully opaque)

## [DoCommand](../DoCommand.md) Execution phase
First, 0.35 seconds are yielded.

From there, all the execution phase occurs in a while loop that goes on while the cursor is still sweeping the bar. How this is determined is a local float starting at 0.0 and growing by a fraction of the game's frametime every iterations until it reaches 1.0 where the loop ends. That faction is `timer`.

Essentially, the local float reports the percentage of completion of the bar from 0.0 to 1.0 and if it reaches 1.0, the execution ends. This local will be referred as t.

The current timepoint is tracked by a local int that starts at 0 and the amount of them by `data[0]` being rounded to int (not truncated).

Here's what happens inside the loop:

- `commandsprites[1]` (the bar) x scale is set to t. NOTE: This will put the bar so that it approximetly aligns with the cursor, but this is slightly off due to the different positioning schemes
- `commandsprites[2]` local x position is set to t * 8.55 (this places the cursor alongside the actual point we are on the bar)
- The following happens as long as we didn't got past the last timepoint:
    - If t * 8.55 (the cursor position) is past the current timepoint + 1.0, the timepoint was not hit (that + 1..0 means there will be a slight delay between when the cursor goes too far and the time the game informs the player):
        - `commandsprites[3 + timepoint index]`.color is set to pure red
        - PlayBuzzer is called
        - The current timepoint is advanced by incrementing it
    - If `presskey` is pressed without hold while we still haven't gotten past the last timepoint, what happens depends on the distance between the timepoint and t * 8.55 (the cursor's position). If it's less than 0.45, `successfullchain` is incremented and `commandsprites[3 + timepoint index]`.color is set to pure yellow. If it's more, it's set to red instead and PlayBuzzer is called instead of incrementing `successfullchain`. In other words, the input counts as a success if it was pressed close enough to the timepoint and failed otherwise
    - The current timepoint is advanced by incrementing it
    - The `ACBeep` sound is played (or `ACReady` if it was the last timepoint)
- t is incremented by a fraction of the game's frametime (that fraction is `timer`)
- A frame is yielded
