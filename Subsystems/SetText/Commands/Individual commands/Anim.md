# Anim

Change the animation state of the party or a specific entity.

## Syntax

````
|anim,entity,anim|
````

## Parameters

### `entity`:  `this` | `party` | `caller` | `parent` | int

The entity to change the animation state:

* `this`: Refers to [tailtarget](../../Notable%20local%20variable/tailtarget.md)'s entity (This only reliably works in [Dialogue mode](../../Dialogue%20mode.md) otherwise, it will take the last value of [tailtarget](../../Notable%20local%20variable/tailtarget.md) which is potentially undefined behavior).
* `party`: Refers to each of the party's entities.
* `caller`: Refers to the `caller` if it is not null. If it is, this command does nothing.
* `parent`: This value is not fully implemented and will cause this command to do nothing, but it was intended to refer to the [Parent](Parent.md)'s entity.
* int: Refers to any other entity index.

Any non int value that isn't in one of the predefined values above will cause an exception to be thrown.

Any other value is treated like a regular [Entity id](../Entity%20id.md). If it resolves null, no changes will happen.

### `anim`: string | int

The animation state to set. The string form must be one of these which have an int form equivalent (this is case sensitive):

|int|string|
|---|------|
|0|`Idle`|
|1|`Walk`|
|2|`Jump`|
|3|`Fall`|
|4|`ItemGet`|
|5|`Angry`|
|6|`Sad`|
|7|`Upset`|
|8|`Happy`|
|9|`Surprized`|
|10|`Flustered`|
|11|`Hurt`|
|12|`Death`|
|13|`BattleIdle`|
|14|`Sleep`|
|15|`Fallen`|
|16|`HurtFallen`|
|17|`WeakBattleIdle`|
|18|`KO`|
|19|`PickAction`|
|20|`WeakPickAction`|
|21|`Woobly`|
|22|`HurtWooble`|
|23|`Chase`|
|24|`Block`|
|25|`SleepFallen`|
|26|`AirTackle`|
|27|`ItemWalk`|
|28|`TossItem`|
|29|`Sit`|
|30|`FakeHurt`|
|31|`Dig`|
|32|`DigMove`|

Any non int value that isn't on this list will be interpreted as an int which will cause an exception to be thrown. Any int value is valid.

## Remarks

This command does nothing when the PauseMenu is up. Otherwise, after the change has been performed (or if the entity was resolved to null) It will yield for a frame.
