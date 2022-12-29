# Alwaysactive

Set the `alwaysactive` field of an [Entity](../../../Data%20format/Entity.md) to a value.

## Syntax

````
|alwaysactive,entity,value|
````

## Parameters

### `entity`: int | string

The [Entity](../../../Data%20format/Entity.md) that this command refers to. The int form is a regular [Entity id](../Entity%20id.md) and it must not resolve to null or an exception will be thrown. The string form can be one of the following when not in battle (if in battle, this is interpreted as an int which will cause an exception to be thrown):

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the `caller`. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [Define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `value`: `true` | `false`

The value to set the `entity`'s `alwaysactive` field. This must be a valid bool or an exception will be thrown.
