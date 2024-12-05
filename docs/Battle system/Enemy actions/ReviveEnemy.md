# ReviveEnemy
A method specifically made for enemy actions that allows to bring back an enemy party member that was in `reservedata` back to `enemydata` with a certain percentage of `hp` left and the optional ability to be able to act immediately after the revive.

```cs
private void ReviveEnemy(int reserveid, float hppercent, bool canmove, bool showCounter = false)
```

## Parameters

- `reserveid`: The `reservedata` index of the enemy party member to revive
- `hppercent`: Determines the `hp` value to leave the revive enemy at. The exact number depends on the specific value (the final `hp` value is always clamped from 0 to the `maxhp`):
    - Above 1.0: The value is the same as the parameter's value
    - Any other values: The value represents the fraction of `hp` to leave the enemy party member with over its `maxhp` ceiled. It should be at least above 0.0 to not have the enemy party member revive with 0 `hp` which may cause undesired behaviors
- `canmove`: If true, the `cantmove` of the enemy party member after the revive will be set to 0 (one actor turn available). If it's false, it will be set to 1 (no actur turns available until the next main turn)
- `showcounter`: If true, [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) will be called to show an hp counter sbowing the new `hp` amount used to revive the enemy party member along with a `Heal` sound playing

## Procedure

- The enemy party member in `reservedata` with index reserveid is removed, but added to the end of of `enemydata`
- The now revived enemy party member has these fields set on them:
    - `hp`: Set accoring to hppercent. See the hppercent parameter desctiption above for details
    - `exp`: 0
    - `money`: 0
    - `cantmove`: 1 (0 instead if canmove is true)
    - `turnssincedeath`: 0
    - `turnsalive`: 0
    - `hitaction`: false
- [ClearStatus](../Actors%20states/Conditions%20methods/ClearStatus.md) called on the now revived enemy party member
- Revive called on the `battleentitiy` of the now revived enemy party member
- If showcounter is true:
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP) on the enemy party member with the ammount being their new `hp` starting at their position + 2.0 in y and ending at Vector3.up
    - `Heal` sound played
- The now revived enemy party member's `alreadycounted` is set to true (so the amount defeated doesn't go up again in the bestiary if they die again)
- [ReorganizeEnemies](../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called with order
