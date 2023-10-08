# Define

Add a name that can be used to resolve an [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) for this SetText call in [Dialogue mode](../Dialogue%20mode.md).

## Syntax

````
|define,id,name|
````

## Parameters

### `id`: int

The [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) to resolve using `name` from now on. This must be a valid [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) or an exception will be thrown when resolving the entity.

### `name`: string

The name that will be used to resolve the [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) `id` from now on. Any string is allowed and it is case sensitive.

## Remarks

This command allows access to a feature some commands supports which is to use the define list to resolve [Entity](../../Entities/Entity.md) by a string key rather than the traditional [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) key. Not all commands supports this feature, check the command's documentation to learn if it is supported. Other commands only supports [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) keys.

In [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode), this command technically works, but it will not clear the define list as this task is done in [Dialogue setup](../Life%20Cycle.md#dialogue-setup).
