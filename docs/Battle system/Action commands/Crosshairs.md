# Crosshairs
A simple Confirm input prompt consisting of a ring that renders on an actor with its matching crosshair moving from a random position towards it which is then repeated multiple times. The goal of the command is to press Confirm when the crosshair falls inside its ring visually and to hit all of the Confirm inputs correctly in a row. If no Confirm inputs is done for any crosshairs or if Confirm is pressed too early, the command is a failure.

## `timer` usage
The amount of frames the first crosshair will move before the command is considered a failure. -1.0 is not a supported value for this action command. This frames amount never changes for all crosshairs if `data[3]` doesn't exist and if it does, it will decrease by `data[3]` after each successful crosshair.

## `data` array

- `data[0]`: The amount of crosshairs to hit for the command to succeed
- `data[1]`: The actor index that the crosshairs will target on. The party depends on the value:
    - If it's 0 or higher, it's an `enemydata` index
    - If it's less than 0, its absolute value - 1 is a `playerdata` index
- `data[2]`: This is both the distance the crosshair will be from the target before heading towards it and it also determines the range of each component of the generated starting position's vector (between - (2 * `data[2]`) and 2 * `data[2]`)
- `data[3]`: The amount of frames that gets removed of the total time the crosshair will move after each sucessful Confirm press. This is optional: no frames are removed if this value doesn't exist

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4
- The `Crosshair` sound is played on `sounds[9]` with 0.5 volume on loop
- `commandsprites` is set to a new array of 2 elements where each is the SpriteRenderer of a new UI object named `crossX` where `X` is the index childed to the `battlemap` with a scale of Vector3.one using the `guisprites[41]` sprite (a white cross with a dot in the center) with a sortorder of 10 + index on layer 15 (`3DUI`). The local position depends on `data[1]`:
    - If it's at least 0, it's the `enemydata`'s battleentity position at index `data[1]` (truncated to int) + (Vector3.up * (the enemy party member's battleentity.`height` - 1.0) + the enemy party member's `cursoroffset`). However, if the enemy party member's [position](../Actors%20states/BattlePosition.md) is `Underground`, it's the enemy party member's battleentity position + (0.0, 0.5, 0.0)
    - Otherwise (meaning `data[1]` is negative), the logic is the same, but the `enemydata` index is the absolute value of `data[1]` - 1 and the local position doesn't change if the enemy party member's [position](../Actors%20states/BattlePosition.md) is `Underground`
- `commandsprites[0]`.sprite is set to `guisprites[93]` (a white crosshair's ring)

## [DoCommand](../DoCommand.md) Execution phase
There's 3 parts to the execution phase: before the `timer` loop which has some initialisation, the `timer` loop itself and after the `timer` loop where there's cleanups.

### Before `timer` loop

- `combo` is set to 0
- The actor's entity corresponding to the command's target is obtained (the enemy party member if `data[1]` is 0 or above, the player party member otherwise)
- The amount of remaining crosshairs is stored in a local int and initialised to `data[0]`
- `caninputcooldown` is set to 10.0
- A starting crosshair position is generated which is done by doing the following:
    - Generate a random vector where each component is between - (2 * `data[2]`) and 2 * `data[2]` (via MainManager.RandomVector)
    - Clamp the magnitude of the vector to be `data[2]` (via calling MainManager.ClampMagnitude from `data[2]` to `data[2]`)
    - Add the `commandsprites[0]`'s position (the crosshair's ring)
    - Repeat if the x/y distance to `commandsprites[0]`'s position (the crosshair's ring) is less than `data[2]`. NOTE: This is an inneficient way to reject any vectors whose z has an absolute value that is too high. It shouldn't have been considered in the vector's generation, but instead of doing that, the sample is rejected

### `timer` loop
This loop goes on until `timer` expires which happens when no Confirm inputs was done for enough time to fail the command. It will be exited early if an early fail happens or if a success happens.

This is what happens inside the loop:

- Both `commandsprites` (the crosshair and its ring) has their z angles incremented by 7x the game's frametime so they continuously rotate
- `commandsprites[1]` (the crosshair) has its position set to a lerpUnclamped from the starting position generated (with the z being the `commandsprites[0]` one which is the crosshair's ring) to `commandsprites[0]`.position (the crosshair's ring) with an unclamped factor of (1.0 - `timer` / initialtimer) * 1.15. This essentially makes the crosshair travel towards its ring, but it will overshoot intentionally the position by 15% which allows the crosshair to go slightly past the center of its ring
- If the distance between both `commandsprites` is less than 0.75, `commandsprites[1]` (the crosshair)'s color is set to pure green. This is only updated for every 3 frames tracked by `timer` floored % 3 being 0
- If Confirm was pressed without hold and `caninputcooldown` expired (normally not for the first 10.0 frames of the loop), the input is handled by either failing or moving on to the next crosshair:
    - If the distance between both `commandsprites` is more than 0.85, the command is failed by calling PlayBuzzer followed by setting `commandsprites[1]` (the crosshair)'s color is set to pure red and breaking out of the `timer` loop early
    - If `wordroutine` isn't in progress, it is stopped and `commandword` is destroyed
    - `wordroutine` is set to a new [ShowSuccessWord](../Visual%20rendering/ShowSuccessWord.md) call for the targetted entity without block or super
    - `commandsprites[1]` (the crosshair)'s color is set to pure white
    - A new starting crosshair position is generated using the same scheme than the first one and assigned to it
    - The amount of remaining crosshairs is decremented
    - If `data[3]` existed, initialtimer is decreased by it (effectively, it removes this much amount of frames for the duration of the next crosshair)
    - `timer` is set to initialtimer
    - `combo` is incremented (this will be used when showing the next success word)
    - The volume of `sounds[9]` (`Crosshair` sound) is saved, then temporarilly set to 0.0
    - The `ACBeep` sound is played with a pitch of 0.9 + `combo` / 10.0 and a volume of 1.0 - 0.075 * `combo`
    - 0.05 seconds are yielded
    - The volume of `sounds[9]` (`Crosshair` sound) is restored
    - `caninputcooldown` is reset to 10.0 frames
- If there's no more remaining crosshair, the command is succeeded by doing the following:
    - Both `commandsprites` are disabled
    - `commandsuccess` is set to true
    - Break out of the `timer` loop early
- If `caninputcooldown` hasn't expired yet, it is decreased by the game's frametime
- `timer` is decreased by the game's frametime
- A frame is yielded

### After `timer` loop

- `sounds[9]` is stopped (`Crosshair` sound)
- `doingaction` is set to false (this will be interpreted as DoCommand being done, but it isn't technically done. This however shouldn't matter in practice because the rest of the coroutine is just cleanup so it is logically done)
- If `commandsuccess` is false (the command was failed), `commandsprites[1]`.color (the crosshair) is set to pure red. Otherwise (the command was succeeded), the `AtkSuccess` sound is played
- 0.5 seconds are yielded
