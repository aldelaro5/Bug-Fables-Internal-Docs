# SetPathJump
Similar than [SetPath](SetPath.md), but after the [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) call, the entity `forcejump` is set to true which makes it jump during its FixedUpdate when moving to the next node if possible.

## Frequency meaning
The same than [SetPath](SetPath.md).

## Additional data
The same than [SetPath](SetPath.md).

## SetUp
This has no logic unlike [SetPath](SetPath.md).

## DoBehavior
The same than [SetPath](SetPath.md), but when the [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) call occurs, it will set the entity `forcejump` to true which makes it jump when possible during the move to the next node.

## Update (Inactive, every 3 frames)
This has no logic unlike [SetPath](SetPath.md).

## EntityControl.[Death](../../EntityControl/Notable%20methods/Death.md)
The same than [SetPath](SetPath.md).

## [RespawnEnemy](../Notable%20methods/RespawnEnemy.md)
This has no logic unlike [SetPath](SetPath.md).