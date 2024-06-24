# Dig
A coroutine made for enemy actions to have the enemy dig and change its [position](../Actors%20states/BattlePosition.md) to `Underground`.

The coroutine should be stored in `checkingdead` as it will be set to null when completed so the caller can yield on it.

```cs
private IEnumerator Dig(int enemyid, int jump)
```

There is an overload available that sends 1 as the jump and relays the enemyid where it sets `checkingdead` to a new coroutine being the one above followed by a frame yield:

```cs
private IEnumerator Dig(int enemyid)
```

## Parameters

- `enemyid`: The `enemydata` index of the enemy party member that will be digging
- `jump`: The amount of jump the enemy party member will do before digging

## Procedure

- Done jump amount of times:
    - [Jump](../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on the enemy party member
    - `Jump` sound plays
    - Yield for 0.5 seconds
    - Yield all frames until the enemy party member's `onground` is true
- `Dig` sound plays
- The enemy party member's `digging` is set to true
- Yield all frames until the enemy party member's `digtime` becomes 29.0 or above
- Yield for 0.25 seconds
- The enemy party member's [position](../Actors%20states/BattlePosition.md) is set to `Underground`
- `checkingdead` is set null indicating the caller that the coroutine completed
