# Movewait

Yield until a specific [entity](../../../Data%20format/Entity.md)'s forcemove lock is released.

## Syntax

````
|movewait,entity|
````

## Parameters

### `entity`: int | string

The [entity](../../../Data%20format/Entity.md) that this command refers to. The int form is a regular [Entity id](../Entity%20id.md) and its restriction for valid values applies. The string form can be one of the following:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the `caller`'s entity. This must not be null or an exception will be thrown.
* Anything else: if the string is in the `Define` list, it will refer to it. Otherwise, the value will be interpreted as the int form.

## Remarks

None.
