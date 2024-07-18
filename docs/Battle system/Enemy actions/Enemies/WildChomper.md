# `WildChomper`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to summon a new [FlyTrap](FlyTrap.md) enemy in the pre move logic when this enemy is the last `enemydata` remaining changes to 51% from 36%
- In the seeds throw move, there is now a 5/10 chance for the property of each DoDamage calls to be [Poison](../../Damage%20pipeline/AttackProperty.md)
- In the seeds throw move, the amount of seeds to throw changes to be random between 2 and 3 inclusive instead of being random between 1 and 3 inclusive
- In the seeds throw move, the amount of frames each seed takes to reach its target changes to 35.0 frames from 41.0 frames
- The vine attack move may hit twice instead of only once

## Move selection
2 moves are possible:

1. A multiple target seeds throw
2. A single target vine attack that may hit multiple times

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

## Pre move logic
The following logic always happen before any move is used if this enemy is the last `enemydata` remaining and a 36% RNG check passes (51% instead if hardmode is true):

- A Vector3 is attempted to be generated that will corresponds to the position to spawn the new enemy. Each attempt may fail, but up to 20 attempts will be done and the first one that fufills some conditions will be used. Each attempts generates a vector where the x component is between 0.25 and 7.0, the y component is 0.0 and the z component is between -0.75 and 0.75. For the location to be accepted, any enemy part member needs to be located at a distance less than its `size` * 1.75. If no suitable locations is found after 20 attempts, the logic will not proceed and move selection is done immeditately
- If a suitable location is found:
    - animstate set to 100
    - `Clomp` sound plays
    - Camera moves to look near this enemy + 1.0 in y
    - Yield for 0.5 seconds
    - `Rumble3` sound plays
    - ShakeScreen called with 0.1 ammount and 0.65 time
    - Yield for 0.65 seconds
    - Camera moves to look near the location
    - `Charge` sound plays
    - DeathSmoke particles plays at the location
    - `checkingdead` is set to a new [NewEnemy](../../Actors%20states/Enemy%20party%20members/NewEnemy.md) call to a summon a [FlyTrap](FlyTrap.md) with the `SeedMini` animation at the location
    - Yield for a frame
    - `enemydata[lastaddedid]` gets some adjustements:
        - z position: 0.0
        - `locktri`: true (prevents them to summon another `FlyTrap`)
        - If `cotunknown` is false, `spritebasecolor` is set to pure cyan
        - `exp`: divided by 2 floored
        - `hp` and `maxhp`: divided by 2 floored
        - If `hologram` is false, `sprite`'s material's color is set to pure cyan
    - [UpdateEntities](../../Visual%20rendering/UpdateEntities.md) called
    - Yield for 0.5 seconds
    - animstate set to 0 (`Idle`)
    - Yield all frames until `checkingdead` is null (the coroutine completed)
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - Yield for 0.5 seconds

## Post move logic
The following logic always happen after a move is used if `hp` is less than `maxhp`:

- animstate set to 0 (`Idle`)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Yield for 0.25 seconds
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by 1 `hp`
- Yield for 0.5 seconds
 
## Move 1 - Seeds throw
A multiple target seeds throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 1 to 3 times inclusive determined randomly (2 to 3 times inclusive instead if hardmode is true), but the call and no further calls happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable returns -1|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|3|null ([Poison](../../Damage%20pipeline/AttackProperty.md) instead if hardmode is true and a 5/10 RNG check passes)|null|`commandsuccess`|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- The property of the DoDamage 1 call is determined. It's null, but if hardmode is true and a 5/10 RNG check passes, it's [Poison](../../Damage%20pipeline/AttackProperty.md) instead
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
    - The `HardSeed` position set to this enemy + (-2.0, 1.25, -0.1)
    - Over the course of 41.0 frames (35.0 frames instead if hardmode is true), `HardSeed` moves to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp. During the second half of the movement, if this isn't the last hit, animstate is set to 101
    - `HardSeed` moves offscreen at 999.0
    - DoDamage 1 call happens
- `HardSeed` gets destroyed
- Yield for 0.5 seconds
- `checkingdead` is set to null which signals the caller that the coroutine completed

## Move 2 - Vine attack
A single target vine attack that may hit multiple times.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|`playerdata[playertargetID]`'s `hp` is above 0. Done once if hardmode is false, from 1 to 2 times if it is true as long as `playerdata[playertargetID]`'s `hp` is above 0|This enemy|The selected `playertargetID`|3 on the first hit, 2 on the second hit|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.65 seconds
- `checkingdead` is set to a new VineAttack coroutine with 2 damageamt, actionid as the callerid without gettarget. The coroutine will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` goes to null

This is what the coroutine effectively ends up doing:

- `Rumble3` sound plays
- ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
- Camera moves to look near `playertargetentity`
- The amount of hits is determined. It is 1 if hardmode is false and if it's true, it's random between 1 and 2
- An array of `Prefabs/Objects/VenusRoot` are instantiated with the amount being the amount of hits to do at (`playertargetentity` x position + a random number between -2.0 and 2.0, -5.0, `playertargetentity` z position). Each instance is set to LookAt the `playertargetentity` and has its initial position incremented by (random between -1.0 and 1.0, 0.0, -0.15)
- An array of Vector3 are instantiated with the amount being the amount of hits to do with each being the normalised direction vector from `playertargetentity` to the matching `Prefabs/Objects/VenusRoot` element
- Yield for 0.75 seconds
- For each hit:
    - `Charge8` sound plays
    - Over the course of 10.0 frames, the matching `Prefabs/Objects/VenusRoot` position lerps to its position + the matching heading direction vector * 5.0. The first time reaching the 5th frame or later when `playerdata[playertargetID]`'s `hp` is above 0, DoDamage 1 call happens before the frame yield
    - Yield for 0.5 seconds
    - If this is the last hit:
        - `Rumble3` sound plays
        - ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
        - Yield for 0.5 seconds
    - The amount of damage to deal on the next hit becomes half of this one's damage floored then clamped from 1 to 99
- Over the course of 60.0 frames, all `Prefabs/Objects/VenusRoot` instances has ther scale lerped to Vector3.zero
- Yield a frame
- All `Prefabs/Objects/VenusRoot` instances gets destroyed
- `checkingdead` set to null which informs the caller that the coroutine is done
