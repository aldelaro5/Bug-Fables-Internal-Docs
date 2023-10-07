# Tail

Set the [tailtarget](../Notable%20states.md#tailtarget) to an [Entity](../../Entities/Entity.md) and optionally change the animation state of the new target [Entity](../../Entities/Entity.md)  with a hide/unhide of the textbox in [Dialogue mode](../Dialogue%20mode.md).

## Syntax

(1)

````
|tail,entity|
````

(2)

````
|tail,entity,transition|
````

## Parameters

### `entity`: `caller` | `parent` | `null` | `this` | int

The [Entity](../../Entities/Entity.md) to set the [tailtarget](../Notable%20states.md#tailtarget) to (the predefined values are case sensitive):

* `this`: Refers to [tailtarget](../Notable%20states.md#tailtarget)'s entity (This only reliably works in [Dialogue mode](../Dialogue%20mode.md) otherwise, it will take the last value of [tailtarget](../Notable%20states.md#tailtarget) which is potentially undefined behavior) Since it is setting the [tailtarget](../Notable%20states.md#tailtarget) to its current entity, this effectively doesn't change it.
* `caller`: Refers to the caller if it is not null. If it is, the value is interpreted as an int which will cause an exception to be thrown.
* `parent`:  Refers to the parent If it is, the value is interpreted as an int which will cause an exception to be thrown.
* `null`:  Refers to the null value. This makes the textbox not point to anyone. Usage of this value mandates that `transition` is `instant` or not specified due to an incompatibility with the [anim](Anim.md) command that will cause an exception to be thrown. See the `transition` parameter section for more details.
* int: Refers to any other [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md).

Any non int value that isn't in one of the predefined values above be interpreted as an int which will cause an exception to be thrown.

### `transition`: `instant` | string | int

The transition method to use when changing the [tailtarget](../Notable%20states.md#tailtarget). `instant` (case sensitive) will prevent the textbox to hide/unhide itself after 0.035 seconds and the command will not have the entity change its animation state via an [anim](Anim.md) command. If this value is not specified, the same behavior happens, but without the animation state change.

Any other value will cause a relay to an [anim](Anim.md) command in the form |[anim](Anim.md),`entity`,`transition`\| at the end of this command's processing. This means that any other value of this parameter must adhere to the specification of the `anim` parameter of the [anim](Anim.md) command which implies `null` is not a valid value for `entity` if the relay happens. This means that to specify `null` as the `entity` parameter, this parameter needs to either be `instant` or not specified.

## Remarks

This command will perform the following operations:

* Resets the state of the [Backtracking](../Related%20Systems/Backtracking.md) system effectively clearing the history, current accumulator and set the current dialogue to the start. 
* Sets the existing [tailtarget](../Notable%20states.md#tailtarget)'s [Entity](../../Entities/Entity.md) to stop talking.
* Obtains the new entity of the [tailtarget](../Notable%20states.md#tailtarget) is obtained, but it will only be set immediately if `entity` is null or `transition` is `instant`. Otherwise, it will first set the [textbox](../Notable%20states.md#textbox) to hide itself and yield for 0.035 seconds. Only after this yield is completed will the [tailtarget](../Notable%20states.md#tailtarget) change entity. An exception to this rule is if the [tailtarget](../Notable%20states.md#tailtarget)'s entity is the same than the one to set it to in which case, the yield and hiding of the textbox are skipped.
* Yields a frame.
* Change the bleep sound and pitch according to the new [tailtarget](../Notable%20states.md#tailtarget)'s entity (the sound is set to null if the new entity is null).
* Yields a frame.
* If `transition` isn't `instant`, set the textbox to unhide itself. 
* If `transition` was specified with a value other than `instant`, relay processing to an [anim](Anim.md) command with the same parameters as this command. If `transition` is `instant`, this will not be done which ends this command's processing.

It is technically possible to use this command in [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode) as long as `transition` isn't `instant` (otherwise, an exception will be thrown). However, doing this can cause undefined behaviors as this command was primarily made to operate in [Dialogue mode](../Dialogue%20mode.md).

This is one of the command supported by [testdiag](Testdiag.md).
