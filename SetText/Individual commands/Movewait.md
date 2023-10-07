# Movewait

Yield until a specific [Entity](../../Entities/Entity.md)'s forcemove lock is released.

## Syntax

````
|movewait,entity|
````

## Parameters

### `entity`: int | string

The [Entity](../../Entities/Entity.md) that this command refers to. The int form is a regular [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) and its restriction for valid values applies. The string form can be one of the following:

* `this`: Refers to the [tailtarget](../Notable%20states.md#tailtarget). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller's entity. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.
