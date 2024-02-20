# EndEnemyTurn
This method is only called by [DoAction](Action%20coroutines/DoAction.md) during the [post action phase](Action%20coroutines/DoAction.md#post-action) if the actionid isn't -555 (a player first strike) and it's an enemy party member's action. It will end an enemy party member's actor turn, just like [EndPlayerTurn](EndPlayerTurn.md), but for enemy party members instead of player party members.

```cs
private void EndEnemyTurn(int id)
```

## Parameters

- `id`: The `enemydata` index whose actor turn should be ending

## Procedure

- If we were not in the enemy party member's `hitaction`, the `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) isn't equipped and the enemy party member's `notired` is false, its `tired` field is incremented (this adds an exhaustion). NOTE: This neglects HARDEST and B.O.S.S's EX mode making this check incorrect as it leaves exhaustion enabled on HARDEST without `DoublePain`, same for EX mode
- If we were not in the enemy party member's `hitaction`, the enemy party member's `cantmove` is incremented (this consumes the actor turn)
- The enemy party member's `hitaction` is reset to false
- Unless `dontusecharge` is true, the enemy party member's `charge` is reset to 0
- RefreshAllData is called which sets `alldata` to a new list which consists of all the `playerdata` followed by all the `enemydata` appended together
- Unless we were `inevent`, [ReorganizeEnemies(true)](../Actors%20states/ReorganizeEnemies.md) is called which removes all dead enemies and orders all `enemydata` by their battleentity's x position
