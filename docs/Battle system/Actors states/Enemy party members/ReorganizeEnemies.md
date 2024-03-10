# ReorganizeEnemies
This is a method that will reorder `enemydata` to a specific list of [enemy](../../../Enums%20and%20IDs/Enemies.md) ids or remove `dead` or [fled](../Enemy%20features.md#fled) enemy party members as well as perform other sanitisation on `enemydata`.

There are 4 overloads available:

(1)
```cs
private void ReorganizeEnemies()
```
Ends up doing (2) with a skip of -1

(2)
```cs
private void ReorganizeEnemies(int skip)
```

(3)
```cs
private void ReorganizeEnemies(bool order)
```
If order is false, it ends up being the same as (1)

(4)
```cs
private void ReorganizeEnemies(int[] order)
```
This reorders `enemydata` instead of sanitizing it

## Parameters

- `skip`: The `enemydata` to skip adding no matter what, -1 to not skip any elements. NOTE: -1 is always sent under normal gameplay
- `order`: This depends on the overload used:
    - For (3), if true, `enemydata` will be ordered by battleentity's x position and the only ones kept are the one where EnemyAlive returns true (the index is valid, the `hp` is above 0, battleentity isn't `dead` or `kill` and battleentity.`deathcoroutine` isn't in progress)
    - For (4), the new desired ordering of the [eneny](../../../Enums%20and%20IDs/Enemies.md) ids. The array must be the same length as `enemydata` or nothing happens

## Procedure
(1) / (2) removes all enemy party members whose `hp` is 0 or below, their index isn't skip and they haven't [fled](../Enemy%20features.md#fled). This is followed by calling SetBEntityIDs which updates all the `enemydata`'s battleentity.`battleid` to their new one.

(3) does the same as (1) if order is false. If it's true, it removes all enemy party members where EnemyAlive returns false which happens if any of the following is true:

- The `enemydata` index is invalid
- The `hp` is 0 or below
- battleentity is `dead` or `kill` 
- battleentity.`deathcoroutine` is in progress

From there, `enemydata` is ordered by battleentity's x position.

(4) doesn't sanitize `enemydata`, but rather reorders it. The order is given by order and it's done by [enemy](../../../Enums%20and%20IDs/Enemies.md) ids. If the length of order isn't the same than `enemydata`, nothing happens.
