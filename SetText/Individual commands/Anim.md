# Anim

Change the [animstate](../../Entities/EntityControl/Animations/animstate.md) of the party or a specific [Entity](../../Entities/Entity.md).

## Syntax

````
|anim,entity,anim|
````

## Parameters

### `entity`:  `this` | `party` | `caller` | `parent` | int

The [Entity](../../Entities/Entity.md) to change the animation state:

* `this`: Refers to [tailtarget](../Notable%20states.md#tailtarget)'s entity (This only reliably works in [Dialogue mode](../Dialogue%20mode.md) otherwise, it will take the last value of [tailtarget](../Notable%20states.md#tailtarget) which is potentially undefined behavior).
* `party`: Refers to each of the party's entities.
* `caller`: Refers to the caller if it is not null. If it is, this command does nothing.
* `parent`: This value is not fully implemented and will cause this command to do nothing, but it was intended to refer to the [parent](Parent.md)'s entity.
* int: Refers to an [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md).

Any non int value that isn't in one of the predefined values above will cause an exception to be thrown.

Any other value is treated like a regular [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md). If it resolves null, no changes will happen.

### `anim`: string | int

The animation state to set. The string form must be a [animstate > Standard animation clips base name](../../Entities/EntityControl/Animations/animstate.md#standard-animation-clips-base-name). Any non int value that isn't on this list will be interpreted as an int which will cause an exception to be thrown. Any int value is valid.

## Remarks

This command does nothing when the PauseMenu is up. Otherwise, after the change has been performed (or if the entity was resolved to null) It will yield for a frame.
