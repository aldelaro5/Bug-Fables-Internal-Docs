# Jump

Make an [Entity](../../../Data%20format/Entity.md) jump up with a configurable velocity.

## Syntax

(1)

````
|jump,entity|
````

(2)

````
|jump,entity,velocity|
````

## Parameters

### `entity`: int | `this`, `caller`, string

The [Entity id](../Entity%20id.md) or designator to jump up. The int form represents an [Entity id](../Entity%20id.md) and it must be a valid int or an exception will be thrown. For other values, they only apply if the caller isn't null otherwise, they will cause an exception to be thrown:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md).
* `caller`: Refers to the caller.
* Anything else: Refers to a [Define](Define.md) if it exists, otherwise, this is interpreted as a regular [Entity id](../Entity%20id.md) which will cause an exception to be thrown.

### `velocity`: float

The vertical velocity of the jump. This must be a valid float or an exception will be thrown. The default value is 10.0.

## Remarks

This uses the entity's Jump method which amongst its function will remove the constraint of an entity to perform the jump if it isn't an item or there is not battle in progress. Additionally, offgroundframes will be set to 20.0 and jumpcooldown to 30.0.
