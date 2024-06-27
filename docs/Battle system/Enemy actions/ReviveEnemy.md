# ReviveEnemy
A method specifically made for enemy actions that allows to bring back an enemy party member that was in `reservedata` back to `enemydata` with a certain percentage of `hp` left and the optional ability to be able to act immediately after the revive.

```cs
private void ReviveEnemy(int reserveid, float hppercent, bool canmove)
```

## Parameters

- `reserveid`: The `reservedata` index of the enemy party member to revive
- `hppercent`: The fraction of `hp` to leave the enemy party member with over its `maxhp` ceiled. This is meant to be higher than 0.0, but at most 1.0
- `canmove`: If true, the `cantmove` of the enemy party member after the revive will be set to 0 (one actor turn available). If it's false, it will be set to 1 (no actur turns available until the next main turn)

## Procedure

- The enemy party member in `reservedata` with index reserveid is removed, but added to the end of of `enemydata`
- The now revived enemy party member has these fields set on them:
    - `hp`: `maxhp` * hppercent ceiled
    - `exp`: 0
    - `money`: 0
    - `cantmove`: 1 (0 instead if canmove is true)
    - `turnssincedeath`: 0
    - `turnsalive`: 0
    - `hitaction`: false
- [ClearStatus](../Actors%20states/Conditions%20methods/ClearStatus.md) called on the now revived enemy party member
- Revive called on the `battleentitiy` of the now revived enemy party member
- The now revived enemy party member's `alreadycounted` is set to true (so the amount defeated doesn't go up again in the bestiary if they die again)
- [ReorganizeEnemies](../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called with order
