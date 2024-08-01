# `ChomperBrute`

## BattleControl.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy is part of the set of the enemies that yields a different clamping on the rewarded amount of exp given their scaled `exp` field when they die (processed by [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)).

If all of these conditions are fufilled, the rewarded amount of exp is clamped to 20 instead of 15 like most other enemies:

- [flags](../../../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)
- `partylevel` is less than 27 (not yet at max rank)
- [Flags](../../../Flags%20arrays/flags.md) 166 is false (not during an EX mode B.O.S.S session)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to summon a new [FlyTrap](FlyTrap.md) enemy in the pre move logic when this enemy is the last `enemydata` remaining changes to 41% from 31%
- In the seeds throw move, the amount of seeds to throw changes to be random between 2 and 3 inclusive instead of being random between 1 and 3 inclusive
- In the seeds throw move, the amount of frames each seed takes to reach its target changes to 35.0 frames from 41.0 frames

## Move selection
2 moves are possible:

1. A multiple target seeds throw
2. A single target head bash

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|4/7|
|2|3/7|

## Pre move logic
The following logic always happen before any move is used if this enemy is the last `enemydata` remaining and a 31% RNG check passes (41% instead if hardmode is true):

- A Vector3 is attempted to be generated that will corresponds to the position to spawn the new enemy. Each attempt may fail, but up to 20 attempts will be done and the first one that fufills some conditions will be used. Each attempts generates a vector where the x component is between 0.25 and 7.0, the y component is 0.0 and the z component is between -0.75 and 0.75. For the location to be accepted, any enemy part member needs to be located at a distance less than its `size` * 1.75. If no suitable locations is found after 20 attempts, the logic will not proceed and move selection is done immeditately
- If a suitable location is found:
    - Camera moves to look near the location
    - This enemy [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) the location
    - animstate set to 100
    - `Clomp` sound plays
    - Yield for a second
    - `Charge5` sound plays
    - animstate set to 104
    - Yield for 0.2 seconds
    - A new sprite object is created rooted at this enemy position + (1.7 if `flip` is true, -1.7 otherwise, 4.6, 0.1) using the `ChomperSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite with a SpinAround that has an `itself` of 20.0 in z
    - Over the course of 70.0 frames, the `ChomperSeed` moves to the location via a BeizierCurve3 with a ymax of 6.0. On the second half of the movement, this enemy animstate is set to 0 (`Idle`) before each frame yield
    - DeathSmoke particles plays at the `ChomperSeed` position
    - `Charge` sound plays
    - `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to a summon a [FlyTrap](FlyTrap.md) with type `FromGroundKeepScale` at the `ChomperSeed` position with cantmove (meaning the summoned enemy can't act on the current main turn and needs to wait on the next one)
    - `ChomperSeed` gets destroyed
    - Yield all frames until `checkingdead` is null (the coroutine completed)
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called

## Move 1 - Seeds throw
A multiple target seeds throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 1 to 3 times inclusive determined randomly (2 to 3 times inclusive instead if hardmode is true), but the call and no further calls happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable returns -1|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup> (target changes for each calls)|2|null|null|`commandsuccess`|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- `flip` set to false
- `checkingdead` is set to a new SpiSeed coroutine call which will set `checkingdead` to null when done
- Yield all frames until `checkingdead` is null

Here is what the coroutine ends up doing:

- animstate set to 100
- Camera moves to look near `partymiddle`
- `Clomp` sound plays
- Yield for 0.25 seconds
- A new sprite object is created rooted offscreen at 999.0 in y using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite with a SpinAround that has an `itself` of 20.0 in z
- The amount of hits is determined. It's random between 1 and 3 inclusive (between 2 and 3 instead if hardmode is true)
- For each hit to do as long as [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1 (result stored in `playertargetID`):
    - `playertargetentity` is manually set to `playerdata[playertargetID]`'s battleentity
    - animstate set to 100
    - Yield for 0.35 seconds
    - animstate set to 102
    - Yield for 0.2 seconds
    - The `HardSeed` position set to this enemy + (-2.0, 2.25, -0.1)
    - Over the course of 41.0 frames (35.0 frames instead if hardmode is true), `HardSeed` moves to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp. During the second half of the movement, if this isn't the last hit, animstate is set to 101
    - `HardSeed` moves offscreen at 999.0
    - DoDamage 1 call happens
- `HardSeed` gets destroyed
- Yield for 0.5 seconds
- `checkingdead` is set to null which signals the caller that the coroutine completed

## Move 2 - Head bash
A single target head bash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- `flip` set to false
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.65, 0.0. -0.1) with a 1.6 multiplier
- Yield all frames until `forcemove` is done
- `Grow2` sound plays with 0.8 pitch
- animstate set to 100
- Yield for a random interval between 0.55 and 0.75 seconds
- animstate set to 105
- `Thump5` sound plays
- `impactsmoke` particles plays at `playertargetentity` position
- ShakeScreen with 0.2 ammount and 1.0 time
- DoDamage 1 call happens
- Yield for 1.25 seconds
