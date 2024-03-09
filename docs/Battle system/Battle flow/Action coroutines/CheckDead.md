# CheckDead
This is an action coroutine that runs at specific points during the battle to check the health of every actors and if any are found dead when they weren't before, the coroutine performs the process to kill them properly mainly by calling [Death](../../../Entities/EntityControl/Notable%20methods/Death.md#death) on their battleentity. The coroutine is usually stored on the `checkingdead` field which tracks whether or not a CheckDead is in progress, but other coroutines may use this field.

NOTE: This field is shared by other coroutines, but as long as care is taken such that no collisions happens, this is fine. Under normal gameplay, this is normally safe.

The coroutine does nothing if `gameover` is in progress. In that case, `checkingdead` is set to null followed by a yield break.

Here's the CheckDead procedure:

- `deadenemypos` is reset to a new list
- `action` is set to true which changes to an [uncontrolled flow](../Update.md#uncontrolled-flow)
- For player party member whose `hp` is 0 or below while their battleentity.`death` is false (meaning this member should be dead, but isn't currently), the player death process occurs (see the section below for more details)
- From there, if there are no longer any player party members with an `hp` above 0 while not `eatenby`, a [DeadParty](../Terminal%20coroutines/DeadParty.md) terminal coroutine is started changing to a [terminal flow](../Update.md#terminal-flow) followed by a yield break which abruptly ends this CheckDead
- If any player party member were killed, SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
- The enemy death checks occurs (see the section below for details)
- If `deadenemypos` isn't empty (meaning at least one enemy party member died or fled during this CheckDead), `extraenemies` isn't empty and there are still player party members with an `hp` above 0 while not being `eatenby`, the `extraenemies` summons procedure is performed (check the section below for details)    
- If there are no longer any enemy party members left while `mainturn` isn't in progress, `mainturn` is set to a new [AdvanceMainTurn](AdvanceMainTurn.md) call which will take care of detecting the terminal case of winning the battle and act accordingly
- Otherwise:
    - A frame is yielded
    - If EnemyDropping reports that all enemy party members's battleentity.`droproutine` isn't in progress while `mainturn` isn't in progress, `action` is set to false which will change to a [controlled flow](../Update.md#controlled-flow) once CheckDead completes
- If we aren't `inevent`, [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called with order
- `checkingdead` is set to null which reports to the rest of BattleControl that CheckDead is no longer in progress

## Player party member death process

- `tired` is set to 0 (removes all exhaustions)
- `charge` is set to 0
- StartDeath is called on the battleentity which starts a [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) process on it
- If `enemy` is true (the player party member died during the [enemy phase](../Update.md#enemies-phase), a [hitaction](../../BattleControl.md#hitactions) or a [delproj advance](AdvanceMainTurn.md#delprojs-advance)), `turnssinedeath` is set to -1 (this is so it gets incremented to 0 on their [AdvanceTurnEntity](../AdvanceTurnEntity.md) later).
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on the player party member
- battleentity.`dead` is set to true (the Death process already does that)
- If the battleentity.[animid](../../../Enums%20and%20IDs/AnimIDs.md) matches the battle's `forceattack`, it is reset to -1 (since the target is no longer valid)

## Enemy party death checks
The following is enclosed in an infinite while loop, but in practice, it will only be iterated a maximum of 2 times.

An enemy party member is considered dead if they have `fled` or their `hp` isn't above 0 while their battleentity isn't `dead` yet. For each that are found:

- The battleentity position is added to `deadenemypos`
- DestroyConditionIcons is called on the battleentity

From there, the dying process depends if the enemy party member `fled` or not.

### Die by fleeing
The following happened if CheckDead found the enemy party member `fled`:

- battleentity position is set to offscreen at (0.0, -1000.0, 0.0)
- If `destroyentity` is true, the battle entity gets disabled

### Die by `hp` not being above 0
The following happens if CheckDead found the enemy hasn't `fled` meaning they were found dead because their `hp` wasn't above 0:

- `charge` is set to 0
- `condition` is reset to a new list
- [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called on the battleentity to unlock its `rigid`
- If `holditem` wasn't -1, [DropItem](../../Actors%20states/Enemy%20party%20members/DropItem.md) is called on the enemy party member with additem true
- If `animid` (the [enemy](../../../Enums%20and%20IDs/Enemies.md) id) is less than the amount of instance.`enemyencounter` (normally 256), `alreadycounted` is false and battleentity.`cotunknown` is false (this isn't a Cave Of Trials's shadow), the corresponding [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md)'s defeated counter is incremented followed by setting `alreadycounted` to true
- The EXP amount is obtained by performing a [GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) call on the `exp`, `fixedexp` and the `animid` (the [enemy](../../../Enums%20and%20IDs/Enemies.md) id) clamped from 0 to instance.`neededexp` unless battleentity is a `hologram` in which case, it's clamped from 0 to 5 instead
- `expreward` is increased by the amount determined above and then clamped from 0 to instance.`neededexp`
- `moneyreward` is increased by the return of GetMoney using `money` which is a uniform random number between 0 and `money` inclusive
- `exp` is set to 0
- If `eventondeath` is -1:
    - [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) is called on the battleentity
    - entity.`spitexp` is set to the amount of EXP determined earlier
    - `money` is set to 0
- Otherwise if `inevent` is false, `inevent` is set to true followed by an [EventDialogue](../EventDialogue.md) starting with `eventondeath` as the id
- If the current enemy party member index is still valid (protects against an edgecase that can involve [EventDialogue](../EventDialogue.md) 9 which involves the `Acolyte` [enemy](../../../Enums%20and%20IDs/Enemies.md)):
    - If [deathtype](../../Actors%20states/Enemy%20features.md#deathtype) is 2, 3, 4 or 5, the enemy party member is added to `reservedata`
    - The `animid` (the [enemy](../../../Enums%20and%20IDs/Enemies.md) id) is added to instance.`lastdefeated`

### Cleanup and `diebyitself` rechecks
The following happens after all checks are done:
        
- UpdateConditionIcons is called which calls UpdateConditionBubbles on the battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- If any enemy was found to be `dead` or `fled` earlier, [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called without skip

Finally, this is where all of this section's logic can be reiterated. It happens if all of the reamining enemy party members have their `diebyitself` set to true. This means that they should die automatically and in order to do so, their `hp` are set to 0 and their `diebyitself` set to false to avoid reentering this case.

By reiterating all the enemies check like this, CheckDead will be forced to detect the death of all remaining enemy party members because their `hp` were just set to 0. This recheck can only happen once making the loop iterating a maximum of 2 times.

## `extraenemies` summons
This section will move the appropriate amount of `extraenemies` to the main `enemydata` array so the ones that died can be replaced when applicable. Here's the process:

- All the enemy party members that died or `fled` during this CheckDead has their `droproutine` stopped then set to null if it was in progress
- For each `deadenemypos` (but not more than the amount in `extraenemies`):
    - `summonnewenemy` is set to true
    - [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) is called with type `Offscreen`, the [enemy](../../../Enums%20and%20IDs/Enemies.md) id being the `extraenemies` at the same index than the `deadenemypos` and the position being the `deadenemypos`
    - 0.5 seconds are yielded
- All frames are yielded as long as `summonnewenemy` is true and EnemiesAreNotMoving reports false (meaning at least one enemy party member has their battleentity is in a [forcemove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove))
- If instance.`map` is the `AbandonedCity` [map](../../../Enums%20and%20IDs/Maps.md) and [flags](../../../Flags%20arrays/flags.md) 400 (temporary slot) is true, all enemy party members have 2 [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) calls on them: one with `AttackUp` for 999999 actor turns and one with `DefenseUp` for 999999 actor turns
- All the corresponding `extraenemies` that were just summoned are removed from `extraenemies`
- A frame is yielded
