# PressKeyTimer
An single input press prompt that lasts a certain amount of frames. The goal of the command is to press the correct input (and only the correct one) on the last 30.0 frames of the command. If no input is taken for the entire duration or if the input is pressed before the last 30.0 frames or if any other inputs recognised by the game is pressed at any point (even if done at the same time than the correct one), the command will be failed. This command has caveats about its UI.

## `timer` usage
The amount of frames that the action command will last in total. No matter what value this is, the last 30.0 frames is the window where a press of the correct input will result in a success. This means that if `timer` is 30.0 or lower, every frames of the action commands will accept a correct input and succeed the command. -1.0 is not supported and will result in an immediate failure of the command.

## `data` array

- `data[0]`: The input the player needs to press on the last 30.0 frames to succeed the command

## Known issues with the UI indicators
There are 2 visual feedbacks that this command provides to assist the player with the timing window:

- The input glyph becomes white only during the last 30.0 frames. It remains gray otherwise
- There's an hexagon ring that progressively constricts itself around the yellow hexagon where the input glyph is located and it's supposed to line up with the yellow hexagon only during the last 30.0 frames

While the input glyph color is always correct and can be relied upon to visualise the timing window, the same cannot be said for the hexagon ring. It seems to only reliably works when the `timer` is 60.0, but anything higher or lower (which the game can do in practice) and it starts to show unexpected visuals. If it's lower, the ring will visually be quite far off from aligning with the yellow hexagon, but still consider inputs to be a success. If it's higher, the ring could be well within the yellow hexagon and still not count.

This is due to the way the ring is scaled: it's done linearly over the initialtimer, but this isn't correct to do. The last 30.0 frames of a `timer` of 60.0 means it's 50% of the way while as for say 45.0, it's 33.33% of the way. This is therefore not something linear, but the game assumes it is which causes visual errors with the ring's timing alignement.

It means that this command can only work visually when `timer` is 60.0. If it's not, the input glyph can still be used, but the ring can be missleading as it's not great to visually indicate the timing.

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to `data[0]`
- `commandsprites` is set to a new SpriteRenderer array with 2 elements:
    - 0: The SpriteRenderer of a new UI object named `hexagon` childed to the `GUICamera` with a local position of (-5.5, 1.5, 10.0), scale of (1.5, 1.5, 1.5) using the `guisprites[42]` sprite (white hexagon) with a sortorder of 0 and a color of pure yellow
    - 1: The SpriteRenderer of a new UI object named `hexagon outline` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (2.0, 2.0, 2.0) using the `guisprites[80]` sprite (a white hexagon ring) with a sortorder of 1
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (0.0, 0.0, 0.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 5
    - parentobj: `commandsprites[0]`

## [DoCommand](../DoCommand.md) Execution phase
All the logic in this command occurs in a while loop that goes on as long as `timer` hasn't expired yet:

- `commandsprites[1]` (the hexagon ring) scale is set to a lerp from 2.0 * Vector3.one to Vector3.one with a factor of 1.0 - `timer` / initialtimer - 0.1. NOTE: This is incorrect, see the section above about the known visual issues to learn more
- `buttons[0]`.`basesprite`.color (the input glyph's color) is set to 7F7F7F (gray) unless `timer` is less than 30.0 in which case, it's set to pure white
- As long as no incorrect inputs was pressed:
    - If the correct input is pressed without hold, `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done). After, one of 2 scenarios is possible (for either, a yield of 0.5 seconds occur after):
        - If `timer` has more than 30.0 frames left (the input was hit too early) or another input was pressed alongside the correct one, the command is failed by calling PlayBuzzer followed by setting `commandsprites[0]`.color (the hexagon) to pure red
        - Otheriwse, the command is succeeded by playing the `Ding2` sound followed by setting `commandsuccess` to true and setting `commandsprites[0]`.color (the hexagon) to pure green
    - Otherwise, if any of the inputs the game recognises was pressed without hold, the command is failed by doing the following (but it will still continue, it just will idle for the rest of the loop iterations):
        - `commandsuccess` is set to false
        - PlayBuzzer is called
        - `commandsprites[0]`.color (the hexagon) is set to pure red
- `timer` is decreased by the game's frametime
- A frame is yielded
