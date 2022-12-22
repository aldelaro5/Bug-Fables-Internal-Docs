# Gettail

An alias of [Tail](Tail.md) where the int form of `entity` refers to the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) of the entity to set the [tailtarget](../../Notable%20local%20variable/tailtarget.md) instead of its [Entity id](../Entity%20id.md).

## Syntax

(1)

````
|gettail,entity|
````

(2)

````
|gettail,entity,transition|
````

## Parameters

Refer to the [Tail](Tail.md) documentation, but the only change is for the `entity` parameter (otherwise, this command behaves exactly like [Tail](Tail.md)).

The int form of `entity` refers to an [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) which allows to set the [tailtarget](../../Notable%20local%20variable/tailtarget.md) without knowing the [Entity id](../Entity%20id.md), but still knowing the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md). 

If it is between 0 and 2 (Team Snakemouth's [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md)), this will be resolve to the corresponding player entity first if it is in the party. 

For any other int (or if the Team Snakemouth [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) isn't in the party), a search will be performed to get every EntityControl present and resolve to the first one that has the desired [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md).

If it isn't found, this can lead to undesired behaviors because it will resolve to a null entity, but unlike specifying `null` as the value, it will not remove the existing [tailtarget](../../Notable%20local%20variable/tailtarget.md). It will remove the bleep however as well as shrink the textbox if `tranistion` isn't instant which effectively leaves a hidden textbox with a dangling [tailtarget](../../Notable%20local%20variable/tailtarget.md).

Any other value for this parameter functions the exact same way than [Tail](Tail.md) and it supports every other value that aren't in the int form.

## Remarks

It should be noted that for searching a non player Team Snakemouth entity, the list is obtained via FindObjectsOfType which does not guarantee its order. This means that if multiple entities with the desired [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) are present, it is ambiguous which one the command will resolve to. For best results, it's best to use this command on a non Team Snakemouth entity only if it's possible to assume only one entity with the desired [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) is present.

By the same token, this method of searching is much slower than the traditional [Entity id](../Entity%20id.md) method offered by [Tail](Tail.md) so this command should only be used if needed and not too often.
