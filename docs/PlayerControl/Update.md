# PlayerControl Update
There are 4 types of Update, only the first one that applies will be done:

- [DashBehvaior](DashBehavior.md)
- Regular movement inputs processing
- [Message](../SetText/Notable%20states.md#message) locked update
- `pause` update

## [DashBehavior](DashBehavior.md)
This update type happens if all of the following conditions are fufilled:

- Not in a `pause`
- Not in a `minipause`
- The [message](../SetText/Notable%20states.md#message) lock is released
- The player is `dashing`

The only thing that happens in this update type is a call to [DashBehavior](DashBehavior.md) which will override the entire inputs processing.

## Regular inputs processing
This update type happens if all of the following conditions are fufilled:

- Not in a `pause`
- Not in a `minipause`
- The [message](../SetText/Notable%20states.md#message) lock is released
- The player isn't `dashing`

In this update type, all movements inputs are processed to among other things assigns the delta fields.

Here's what happens in this update type in order.

### Setup

- `delta` is set to Vector3.zero (no movement for now as it will get accumulated later)
- `lastaxis` is set to a new Vector3 where the x component is Joystick 0 (horizontal) and the z component is Joystick 1 (vertical). The y component is left at 0.0

### Movement inputs
This is where the main movements inputs are processed. It only happens if `lockkeys` is false.

The way it works is 2 pairs of inputs are processed in a mutually exclusive way. 2 and 3 (left/right) goes firs and they will have at most 1 of the 2 inputs processed with the former taking priority if both are detected. After left or right is processed, 0 and 1 (up/down) goes after in the same manner with up taking priority. Also note that an input must be held here in order to be detected: one frame tap inputs aren't processed.

By the end of this process, it ends up doing the following depending on the inputs processed:

- `delta` gets increased by a vector
- entity.`flip` and `trueflip` gets updated
- entity.`backsprite` gets updated

The `delta` and entity.`flip` updates depends on if we are processing digital or analogue inputs. The latter is done when `lastaxis`'s matching component (x for left/right, z for up/down) isn't 0.0 meaning there is analogue inputs to process and they will take precedence if detected. Here's what changes if processing analogue inputs (they don't happen for digital inputs):

- The vector `delta` is increased by gets multiplied by the absolute value of `lastaxis`'s matching component so it can scale it down depending on how much the analogue input is pushed
- For left or right input, entity.`flip` only gets updated if any of the following conditions are fufilled (the value depends on the input, also, it doesn't affect `trueflip` which always gets updated to the same value matching the input):
    - The player is `flying`
    - The player is in `shield`
    - The absolute value of `lastaxis`.x is higher than 0.1
- For the up input specifically, entity.`backsprite` update logic changes, see the tables notes below for details

Here's what the inputs does in regards to all of the above:

|Input|`delta` vector added|New entity.`flip` and `trueflip`|New entity.`backsprite`|
|----:|-----|-----|-----|
|2 (left)|-`CamDir`.right.normalized|false|false|
|3 (right)|`CamDir`.right.normalized|true|false|
|0 (up)|`CamDir`.forward.normalized|Left unchanged|Depends on conditions<sup>1</sup>|
|3 (down)|-`CamDir`.forward.normalized|Left unchanged|false|

1: The update logic gets more complex in specific conditions. The value only gets updated if any of the following conditions are fufilled (left unchanged if none applies):

|Condition|Value set if fufilled|
|--------|--------------------|
|The player is `flying`|false|
|The player is in `shield`|false|
|`lastaxis`.z is 0.0 (digitial up input)|true|
|The absolute value of `lastaxis`.z is higher than 0.1 (analogue up input that is pushed hard enough)|true|

This is done like this because it should be possible to very slightly push the up analogue input such that it should be enough to slowly move forward, but not enough to have the player's `backsprite` be rendered.

### `idletime` updates
`idletime` gets updated depending on the conditions below:

- `delta` is Vector3.zero (no movement)
- Input.`anykey` is false (no inputs is being pressed)
- `canpause` is true

If all of them are fufilled, then the value is increased by `framestep` unless it has reached 1000.0 or more where it's left unchanged.

Otherwise, the value is set to 0.0 since the player isn't idling anymore.

### `walkdelta` computation
`walkdelta` gets set to a value depending on MainManager.`analog`.

- 0 (OFF): Set to `delta`.normalized * entity.`speed`
- 1 (LOW): Set to `delta`.normalized * entity.`speed` * a number that depends on the absolute value of the magnitude of `delta`. If it's less than 0.4, then this number is 0.6 (so it's decreased by 40%) and 1.0 otherwise (so the number doesn't change the vector)
- 2 (FULL): Set to `delta` magnitude clamped to 1.0 * entity.`speed`. NOTE: Clamping the magnitude like this is the same than normalizing so it is effectively the same than if the setting was set to 0 (OFF)

### `delta`, `lastdelata` and `movecd` updates
These 3 fields are set in the following ways:

- `delta`: Set to its normalized version * entity.`speed`
- `lastdelta`:
    - If `delta` is not Vector3.zero (there is movement), the value is set to `delta`.normalized
    - Otherwise, it's left unchanged
- `movecd`:
    - If `delta` isn't Vector3.zero (there is movement), the value is  increased by `framestep` except if it's 10.0 or above where it's left unchanged
    - Otherwise, it's set to 0.0 (since there's no movement)

### `detect` alignement
The entity's `detect` is set to LookAt its position + `lastdelta` so it becomes aligned to where the entity is going.

### Other inputs
Finally, if `lockkeys` is false, [GetInput](GetInput.md) is called which processes any other inputs than movement.

## [Message](../SetText/Notable%20states.md#message) locked update
This update type only happens if the `message` lock is grabbed.

The only thing that happens is [StopMoving](../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) is called on the entity with targetstate being its current `animstate`. Essentially, it forces the x/z velocity to be zeroed out constantly during a `message` lock.

## `pause` update
This update type only happens if all of the following conditions are fufilled:

- We are in a `pause`
- The entity of this PlayerControl's [animid](../Enums%20and%20IDs/AnimIDs.md) is 1 (`Beetle`)
- The [animstate](../Entities/EntityControl/Animations/animstate.md) of the entity is 100 (this is the animation when using the Horn Slash field skill)

The only thing that happens is [CancelAction](Actions/CancelAction.md) is called.
