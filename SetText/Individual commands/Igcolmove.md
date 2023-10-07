# Igcolmove

Ignore all collision with the player to a specific [Entity](../../Entities/Entity.md) during this SetText call in [Dialogue mode](../Dialogue%20mode.md).

## Syntax

````
|igcolmove,entity|
````

## Parameters

### `entity`: int

The [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) or designator to ignore collisions with the player. The int form represents an [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) and it must be a valid int or an exception will be thrown. If the entity resolves to null, this command does nothing:

* `this`: Refers to the [tailtarget](../Notable%20states.md#tailtarget).
* `caller`: Refers to the caller.
* Anything else: Refers to a [define](Define.md) if it exists, otherwise, this is interpreted as a regular [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md) which will cause an exception to be thrown.

## Remarks

This command does nothing if the player is null.

Specifically, the collisions ignored concern the [Entity](../../Entities/Entity.md)'s ccol, pusher, scol and boxcol if each exists. This also set the hitwall field of the player's [Entity](../../Entities/Entity.md) to false.

In [Dialogue mode](../Dialogue%20mode.md), the collisions are restored in the [Dialogue Cleanup](../Life%20Cycle.md#dialogue-cleanup) phase if the player is still not null and the game isn't in an event, but in [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode), they won't be restored after the SetText call.
