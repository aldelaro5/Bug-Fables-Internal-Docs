# ChangePosition
A coroutine specifically made to be used inside an enemy action which allows to smoothly change the [position](../Actors%20states/BattlePosition.md) of the enemy party member. When the coroutine completes, `checkingdead` is set to null to inform the caller so it can yield on it.

Changing to `Flying` or `Ground` is supported as well as changing from `Underground` to `Ground` (but not `Flying`). Changing to `Underground` isn't supported (use [Dig](Dig.md) for that instead).

```cs
private IEnumerator ChangePosition(int entityid, BattlePosition position)
```

## Parameters

- `entityid`: The `enemydata` index that will change its `position`
- `position`: The new BattlePosition to set. While any values will end up being set, not all of them will act smoothly, see the section above for details. If this new position is the same than the current one, this coroutine does nothing except setting `checkingdead` to null

## Procedure
If the new position is the same than the enemy party memebr's `position`, the coroutine does nothing except setting `checkingdead` to null to inform the caller that it is completed.

Otherwise:

- Over the course of 20.0 frames, the enemy party member's `height` changes from `initialheight` to `minheight` (from `minheight` to `initialheight` instead if the new position is `Flying`) via a lerp
- If the new position is `Flying`:
    - The enemy party member's `bobrange` and `bobspeed` are set to their `startbf` and `startbs` respectively
    - The enemy party member's `height` is set to their `initialheight`
- Otherwise:
    - If the current position is `Underground`:
        - The enemy party member's `digging` is set to false
        - Yield all frames until the enemy party member's `digtime` reached 0.0 or below
        - Yield for 0.25 seconds
    - The enemy party member's `bobrange` and `bobspeed` are set to 0.0
    - The enemy party member's `height` is set to their `minheight`
- The enemy party member's `position` is set to the new position
- `checkingdead` is set to null informing the caller that this coroutine completed
