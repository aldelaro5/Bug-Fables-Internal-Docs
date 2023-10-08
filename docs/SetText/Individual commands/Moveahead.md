# Moveahead

Instruct the game to start a forcemove on an [Entity](../../Entities/Entity.md) where the destination is relative to its current position with a configurable speed, start and stop state.

## Syntax

(1)

````
|moveahead,entity,x,y,z|
````

(2)

````
|moveahead,entity,x,y,z,multiplier|
````

(3)

````
|moveahead,entity,x,y,z,multiplier,state|
````

(4)

````
|moveahead,entity,x,y,z,multiplier,state,stopstate|
````

## Parameters

### `entity`: int | string

The [Entity](../../Entities/Entity.md) that this command refers to. The int form is a regular [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) and its restriction for valid values applies. The string is only applicable if caller isn't null and it will be interpreted as the int form otherwise. This form can be one of the following:

* `this`: Refers to the [tailtarget](../Notable%20states.md#tailtarget). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `x`: float

The x offset to move the [Entity](../../Entities/Entity.md) to. This must be a valid float or an exception will be thrown.

### `y`: float | `null`

This parameter value is ignored as its value is overridden to 0.0, but it must have a value for the command to be properly parsed. If it is not `null`, it must be a valid float or an exception will be thrown.

### `z`: float

The z offset to move the [Entity](../../Entities/Entity.md) to. This must be a valid float or an exception will be thrown.

### `multiplier`: float

The multiplier to apply to the regular speed at which the [Entity](../../Entities/Entity.md) will be moved. This must be a valid float or an exception will be thrown. If no value is specified, 1.0 is the default value.

### `state`: int

The animation state to set the [Entity](../../Entities/Entity.md) when the forcemove will start. This must be a valid int or an exception will be thrown. If no value is specified, 1 is the default value.

### `stopstate`: int

The animation state to set the [Entity](../../Entities/Entity.md) when the forcemove is completed. This must be a valid int or an exception will be thrown. If no value is specified, 0 is the default value.

## Remarks

This command is very similar to [move](Move.md), but it has 2 main differences. The first one is the `y` parameter is ignored, its value will always be 0.0. The second one is the final position to move the [Entity](../../Entities/Entity.md) isn't directly given, but rather is the result of adding (`x`,0,`z`) to the current entity's position. This allows to move the entity using a vector relative to its position rather than directly specify the new position.

If for whatever reason, the [entity](../../Entities/Entity.md) cannot complete its forcemove after a while, a failsafe will trigger after it times out. This failsafe will instantly teleport the entity to its destination with smokes particles and end the forcemove. The specific time for a player entity is 250 frames or 375 frames during an event and if it is not a player entity, it's 500 frames when not in battle (or during a battle, but a checking dead is running) and 225 frames otherwise. The amount is doubled if extratimer is enabled on the [entity](../../Entities/Entity.md).

It should be noted however that due to a regression, this failsafe does not work in 1.1.1 and any situations that would have triggered it may results in a softlock instead. This issue was fixed in 1.1.2.
