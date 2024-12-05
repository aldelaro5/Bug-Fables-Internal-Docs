# GetTargetList
Refresh all enemy party members's [BattlePosition](../BattlePosition.md) and obtains an array containing the `enemydata` index whose position are contained in a provided list.

```cs
private int[] GetTargetList(List<BattlePosition> pos)
```

## Parameters

- `pos`: A list of [BattlePosition](../BattlePosition.md) to search for

## Procedure

- RefreshEnemyPos is called which checks all enemy party members whose `hp` is above 0, whose [cantfall](../Enemy%20features.md#cantfall) is false and whose [position](../BattlePosition.md) is `Ground` or `Flying`. If the enemy battleentity.`height` is above battleentity.`minheight` + 0.5, the [position](../BattlePosition.md) is set to `Flying`, `Ground` otherwise
- Check the `position` of every `enemydata` and if it is contained in the sent pos list, it is added to a list
- The list ToArray is returned
