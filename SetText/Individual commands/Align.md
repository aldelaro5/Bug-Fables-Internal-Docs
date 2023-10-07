# Align

Move the party and Chompy's [Entity](../../Entities/Entity.md) if present to be aligned in a row to the right or left of the caller via a forcemove.

## Syntax

(1)

````
|align,direction,hide|
````

(2)

````
|align,direction,hide,offsetorflipback|
````

## Parameters

### `direction`: `left` | `near` | `front`

The logic to use to determine whether to move the party to the right or left of the caller. Any other value will cause the party to move to the right.

Here are the logic of the different values:

* `left`: Will always move to the left.
* `near`: Moves to the relative position of the party and the caller from the camera's viewpoint. If the party is horizontally positioned on the left side of the caller, they will move to the left of them. If the party is on the same horizontal position or positioned on the right side of the caller, they will move to the right of them.
* `front`: Moves towards the direction the caller's sprite is looking (more specifically, it depends if the caller's sprite is flipped or not). 

### `hide`: `true` | var

Whether to toggle the shrink/reveal animation of the textbox if it exists in [Dialogue mode](../Dialogue%20mode.md). Any other value than `true` will indicate to not do so while `true` indicates to do it. If the textbox does not exist, this parameter does nothing.

### `offsetorflipback`: float | `true` | `false`

Set how far the entities should be from the caller (offset) or set whether or not the caller will face towards the party after the alignement is complete (flipback). Any other value will be interpreted as the bool form and cause an exception to be thrown. It is not possible to specify both settings at once: only one can be specified at a time. The default value for the offset is the radius of the caller's ccol * 3.0 + 0.1 and the default value for flipback is `false`.

The float form can only be used if the first char of the value is between `0` and `9` which means negative float are not supported and will be interpreted as the bool form which will cause an exception to be thrown. Likewise, the value as a whole needs to be a valid float or an exception will be thrown.

The bool form must be `true` or `false` otherwise, an exception will be thrown.

## Remarks

This command does nothing if the caller is null.

### Setup

First, the `overridefollower` flag is set to true while making sure to save its old value for restoring it later. The same is done for Chompy's `following` target if she is present. The followers are then teleported 2.5 away from the player. The command then gathers all the parameters and disables the caller's pusher collider.

### Party's forcemove

Then, the player entities are setup for the forcemove. Their `forcejump` are set to true except for the lead and `backsprite` to false. The forcemove is then setup so that each player entity will move at the desired position determined via `direction` (multiplied by the offset) * 1.3 * index  where the index is the order of the player entities starting from 0 and offset is the float form of `offsetorflipback` or its default value. This essentially will move the party so they are 1.3 apart at the position which is the `direction` combined with the offset. This is all done without speed multiplier or consideration of the y position. The [animstate](../../Entities/EntityControl/Animations/animstate.md) during the move are the default Walk and Idle when done. After the move, the player entities will face towards the caller. Finally, the collisions between the `ccol` of the player entities with the `ccol` of the caller are disabled.

### Chompy's forcemove

For Chompy's forcemove (if she is present), this works a bit differently. She is set to follow the player entity furthest away from the lead at the direction determined with `direction` (multiplied by the offset) * 0.95 towards the front of the camera * −0.45 (which means she will be slightly to the front compared to the party). Other than that, everything is the same (except chompy has no backsprite so this field isn't set).

After everyone's forcemove is setup, this is where the textbox's hide/reveal animation is toggled if `hide` was true. 

### Monitoring the party's forcemove

After, the command will monitor when the party stops moving. Every frame where the move is still ongoing, every player entity's `backsprite` is set to false, every entity whose forcemove is complete will be set to face the caller and a frame is yielded. There is also an additional failsafe that can trigger here: If it's been more than 300 frames that the forcemove is ongoing, the position of each player entities will be set to their target. This failsafe is an addition to the regular forcemove one described in the [move](Move.md) command. The lowest timeout will be used in practice.

### Monitoring the Chompy's forcemove

Once the player entities moves are complete, If Chompy is present and is still moving, the command will yield until she isn't. The failsafe however still applies here and the framecount is cumulated meaning if the party's move have triggered the failsafe, it will be immediately triggered for Chompy as well. It also means that Chompy can trigger it alone even if the party does not trigger it. Once done, her ccol collision with the caller's are enabled back, she will be facing the caller and her anim state reset to Idle.

### Cleanups

After everyone is done moving, the shrink animation of the textbox is toggled back to where it was if `hide` was true. 

The player entities's `ccol` are enabled back, their `backsprite` are set to false again and they are set to face the caller for the last time. The caller's pusher collider is enabled back and `overridefollower` is restored to its original value.

Finally, skiptext from [Text advance](../Related%20Systems/Text%20advance.md) is set to false and the flipback is performed if `offsetorflipback` was true by having the caller face towards the player. Chompy's following target is restored to its older value if she is present and initialflip is set to the player's flip.
