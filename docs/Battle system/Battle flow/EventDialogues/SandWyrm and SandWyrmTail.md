# [SandWyrm](../../Enemy%20actions/Enemies/SandWyrm.md) and [SandWyrmTail](../../Enemy%20actions/Enemies/SandWyrmTail.md) EventDialogues
This event dialogue occurs either by `SandWyrm` or `SandWyrmTail`'s [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) which is assumed to be configured to this EventDialogue. Each have their own EventDialogue, but they are documented in the same page as they are closely related.

## EventDialogue 13
This EventDialogue is meant to trigger as the [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) of `SandWyrmTail`. 

Due to special logic that happened in [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck), it is guaranteed that a `SandWyrm` and a `SandWyrmTail` are part of the enemy party (the `SandWyrm` couldn't have died because if they did, the `SandWyrmTail` would have been killed too, check the EventDialogue 14 details below for why).

This EventDialogue allows to [stop](../../Actors%20states/IsStopped.md) `SandWyrm` temporarilly when `SandWyrmTail` dies.

- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on the `SandWyrm`
- `SandWyrmTail` has its `deathroutine` set to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call on it with activatekill
- `SandWyrm` animstate set to 105
- `TidalDeath` sound plays with 1.125 pitch
- Yield for a second
- Yield for 0.1 second
- DeathSmoke particles plays at the `SandWyrm` position + (-1.5, 0.0, -0.1)
- ShakeScreen called with an ammount of 0.05 and 0.25 time
- `Death3` sound plays
- Yield for 0.25 seconds
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Flipped](../../Actors%20states/BattleCondition/Flipped.md) condition on `SandWyrm` for 2 main turns
- `SandWyrm`'s `size` set to 2.75
- `SandWyrm`'s `cursoroffset` set to (-2.5, 1.5, 0.0)
- Yield all frames until `SandWyrmTail`'s `deathcoroutine` completed
- `SandWyrm`'s VenusBattle is SetUp with itself as the target, but null as the parent
- `SandWyrm`'s `size` set to 2.75
- `SandWyrm`'s `basestate` set to 15 (`Fallen`)
- [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called

## EventDialogue 14
This EventDialogue is meant to trigger as the [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) of `SandWyrm`. It thoroughly kills `SandWyrmTail` and themselves:

- The enemy party member of the [SandWyrmTail](../../Enemy%20actions/Enemies/SandWyrmTail.md) is obtained (it's possible it doesn't exist if the `SandWyrm` was called when still [Flipped](../../Actors%20states/BattleCondition/Flipped.md))
- ShakeScreen called with an ammount of 0.05 and 0.85 time
- `TidalDeath` sound plays
- `SandWyrm` animstate set to 18 (`KO`)
- `SandWyrm`'s [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to -1 (needed for this EventDialogue to not trigger again when the `SandWyrm` dies)
- If the `SandWyrmTail` exists in `enemydata`:
    - Their `hp` is set to 0
    - Their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) is set to -1 (so their EventDialogue don't trigger)
- Yield for a second
- Yield for 1.5 seconds
- If the `SandWyrmTail` exists in `enemydata`:
    - [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) is called on them with activatekill
- `SandWyrm` has its `deathroutine` set to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call on it with activatekill
- Yield all frames until `deathroutine` is null (the `SandWyrm` death finished)
- `checkingdead` is set to a new [CheckDead](../Action%20coroutines/CheckDead.md) call which will kill both `SandWyrm` and `SandWyrmTail`
- Yield all frames until `checkingdead` is null (the CheckDead completed)
- If there was no enemy party member alive with an `hp` above 0 at the start of this action and `cancelupdate` is false (haven't yet switched to a [terminal flow](../Update%20flows/Terminal%20flow.md)), [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) is called with addexp and no skipids to end the battle
