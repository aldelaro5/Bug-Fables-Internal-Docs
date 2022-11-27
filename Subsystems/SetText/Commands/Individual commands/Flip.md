# Flip

Toggle or set the flip state of an entity or the party.

## Syntax

(1)

````
|flip,party|
````

(2)

````
|flip,party,newflip|
````

(3)

````
|flip,entity|
````

(4)

````
|flip,entity,newflip|
````

## Parameters

### `party`

This value indicates to operate with syntax (1) / (2) and will have each party's flip state be affected. Any other value will be interpreted as `entity`.

### `entity`: int | string

The entity to set its flip state. The int form is a regular [Entity id](../Entity%20id.md) and its restriction for valid values applies. The string is only applicable if `caller` isn't null and it will be interpreted as the int form otherwise. This form can be one of the following:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the `caller`. This must not be null or an exception will be thrown.
* Anything else: if the string is in the `Define` list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `newflip`: `true` | `false`

The new flip state to set `entity` or the party. Any other value than `true` or `false` will throw an exception.

## Remarks

Syntax (1) and (3) toggles the flip state while (2) and (4) sets it to a specific one.
