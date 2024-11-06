# DashBehavior
This method is only called by Update when the player is `dashing`. It completely overrides any inputs processing that Update normally does to handle the `dashing` which can only be initiated by a `Beetle`'s [DoActionTap](Actions/DoActionTap.md) with a double tap.

This page documents what the method does by sections.

## Setup

- `lastaxis` is set to (JoyStick(0) which is horixontal, 0.0, JoyStick(1) which is vertical)
- `dashtarget` is set to its normalized version

## Movement input processing
The way it works is 2 pairs of inputs are processed in a mutually exclusive way. 2 and 3 (left/right) goes first and they will have at most 1 of the 2 inputs processed with the former taking priority if both are detected. After left or right is processed, 0 and 1 (up/down) goes after in the same manner with up taking priority. Also note that an input must be held here in order to be detected: one frame tap inputs aren't processed.

By the end of this process, it ends up doing the following depending on the inputs processed:

- `dashtarget` gets increased by a vector
- entity.`flip` gets updated (only for left/right inputs)

The `dashtarget` depends on if we are processing digital or analogue inputs. If processing analogue inputs, the vector `dashtarget` is increased by gets multiplied by the absolute value of `lastaxis`'s matching component so it can scale it down depending on how much the analogue input is pushed

Here's what the inputs does in regards to all of the above:

|Input|`dashtarget` vector added|New entity.`flip`|
|----:|-----|-----|
|2 (left)|-`CamDir`.right.normalized|false|
|3 (right)|`CamDir`.right.normalized|true|
|0 (up)|`CamDir`.forward.normalized|Left unchanged|
|3 (down)|-`CamDir`.forward.normalized|Left unchanged|

## Movement

- `movecd` is increased by `framestep` (it is assumed that dashing is constant movement)
- `dashdelta` is set to a lerp from the existing one to `dashtarget`.normalized * 4.0 with a factor of `framestep` * 0.025. This gets a vector that slightly goes towards `dashtarget` with a magntude of 4.0, but only goes slightly towards it so any change of direction will be delayed when turning
- The entity [Move](../Entities/EntityControl/Notable%20methods/Move.md) to its position + `dashdelta` at 1.0 multiplier and the state depends on [flag](../Flags%20arrays/flags.md) 39 (obtained upgraded dash): if it's true, it's 117 and if it's false, it's 116
- entity.`backsprite` is set to false

## Angling and entity.`detect` alignement

- [DetectDirection](../Entities/EntityControl/EntityControl%20Methods.md#detectdirection) called on the entity to make entity.`detect` LookAt the player position + `dashdelta` so `detect` gets aligned with where the player is going
- entity.`sprite` y angle is set to entity.`detect` y angle
- The new entity.`sprite` local y angle is determined. TODO: figure out the angling math, it looks complex
- `tbox` angles are set to entity.`detect` angles if it exists

## Last dash effects

- entity.`forcemove` is set to false
- `BeetleDash` sound plays on the entity at 0.5 pitch if it wasn't playing already on entity.`sound`

## Y velocity reset
If the following conditions are all fufilled, the entity.`rigid` y velocity is reset to 0.0:

- entity.`offgroundframes` is higher than 20.0 (it's been more than 20.0 frames since the entity was in the air)
- entity.`rigid` gets higher than entity.`jumpheight`

This is to prevent a case where some stray velocity set could cause the dash to launch the entity too high and too suddenly in the air.

## [StopDash](StopDash.md) cases

There are 2 cases in which [StopDash](StopDash.md) gets called (the first one that applies is the one done):

- Either we are in an instance.`pause` or entity.`hitwall` is true while `boulderbreak` is 0.0 or below (it's inactive). In that case, StopDash is call with a wall of entity.`hitwall`
- Either input 4 (Jump/Interact) or 5 (Ability) is pressed. In that case, StopDash is called with a wall of false
