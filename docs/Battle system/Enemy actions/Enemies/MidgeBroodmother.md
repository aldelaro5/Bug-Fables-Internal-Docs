# `MidgeBroodmother`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- On the start of the first actor turn that [HPPercent](../../Actors%20states/HPPercent.md) gets lower than 0.5, a phase transition will happen that decrements this enemy's `cantmove` and sets the `moves` of this enemy to 2. The overall effect is this enemy now gets 2 actor turns per main turns including the current one
- In the dash attack move, the total amount of frames the enemy moves towards the target and past it changes to 37.0 from 48.0
- In the electric ball projectile throw move, the odds to use the [Delayed Projectile](../../Actors%20states/Delayed%20projectile.md) version when there are no `delprojs` changes to 41% from 33%
- In the electric ball projectile throw move (multiple hits version), the amount of projectiles to throw changes to be random between 2 and 3 instead of always being 2
- In the electric ball projectile throw move (multiple hits version), the yield time before each [Projectile](../../Damage%20pipeline/Projectile.md) calls changes to 0.5 seconds instead of 0.65 seconds
- In the electric ball projectile throw move (multiple hits version), the speed of each projectile changes to 40.0 from 50.0
- In the electric ball projectile throw move ([Delayed Projectile](../../Actors%20states/Delayed%20projectile.md) version), the framespeed changes to 60.0 from 80.0
- In the electric ball projectile throw move ([Delayed Projectile](../../Actors%20states/Delayed%20projectile.md) version), `cantmove` gets decremented at the end which grants an additional actor turn to this enemy
- In the enemy summon moves, `cantmove` gets decremented at the end which grants an additional actor turn to this enemy

## Move selection
3 moves are possible:

1. A single target dash attack
2. An electric ball projectile throw that either hits multiple times on multiple targets or is added as a [Delayed projectile](../../Actors%20states/Delayed%20projectile.md)
3. Summons a new [Midge](Midge.md) enemy

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/4|
|2|2/4|
|3|1/4|

However, move 3 won't be used if selected when there's already more than 2 enemy party members or if no suitable location is found to spawn the enemy after 20 attempts. If this happens, a continue directive is issued on the action loop which restarts the entire action logic in the same actor turn (this includes the pre move logic).

## Pre move logic
The following logic always happen at the start of the actor turn:

- `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to set the [position](../../Actors%20states/BattlePosition.md) of this enemy. The position to set is `Flying` unless move 2 (electric projectile throw) is slated for usage where the position is set to `Ground` instead
- Yield all frames until `checkingdead` is null (the coroutine completed)
- If hardmode is true, [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and `moves` is 1 (meaning the phase transition hasn't happened yet), a phase transition occurs which notably sets `moves` to 2:
    - ShakeSprite called with 0.1 intensity and 30.0 frametimer
    - animstate set to 103
    - `Charge7` sound plays
    - Yield for 0.5 seconds
    - `StatUp` sound plays
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
    - Yield for 0.5 seconds
    - `cantmove` is decremented which gives a free actor turn on this enemy in the current main turn
    - `moves` is set to 2 meaning this enemy now gets 2 actor turns per main turns (including the current one since `cantmove` was just decremented)

## Move 1 - Dash attack
A single target dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 103
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- `BMCharge` sound plays
- Over the course of 41.0 frames, this enemy moves + 2.0 in x via a BeizierCurve3 with a ymax of -2.0
- ShakeSprite called with 0.1 intensity for 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 100
- `BMCharge2` sound plays
- Over the course of 48.0 frames (37.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` - (distance between this enemy and `playertargetentity`) in x and -0.1 in z via a BeizierCurve3 with a ymax of 0.35 - `height`. On the first frame that this enemy x position gets lower than `playertargetentity` x position:
    - ShakeScreen called with 0.1 ammount and 0.5 time
    - DoDamage 1 call happens
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Over the course of 31.0 frames, this enemy moves from (15.0, 0.0, 0.0) to startp via a lerp

## Move 2 - Electric projectile throw
An electric ball projectile throw that either hits multiple times on multiple targets or is added as a [Delayed projectile](../../Actors%20states/Delayed%20projectile.md) without damages (but gains an actor turn if hardmode is true where damages may be dealt).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Happens only if the multiple hits version of the move is used which always happen if `delprojs` isn't empty or if it is, it's used if a 67% RNG check passes (59% instead if hardmode is true). When this call happens, it is done 2 times (from 2 to 3 times instead if hardmode is true), but each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup> (target changes for each calls)|A new `Prefabs/Objects/EnergySphere` GameObject rooted positioned at this enemy + (1.15, 2.8, -0.1) with `NoMapColor` tag|50.0 (40.0 instead if hardmode is true)|Random integer between 3 and 5 inclusive then cast to float|`SepPart@3@1,keepcolor`|`Stars`|`PingShot`|null|Vector3.zero|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Happens only if the delayed projectile version of the move is used which only happen if `delprojs` is empty and a 33% RNG check passes (41% instead if hardmode is true)|A new `Prefabs/Objects/EnergySphere` GameObject rooted positioned at (0.0, 20.0, 0.0) with `NoMapColor` tag|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|Random between 2 and 3|0|[Numb](../../Damage%20pipeline/AttackProperty.md)|80.0 (60.0 instead if hardmode is true)|This enemy|null|null|`Fall2`|

### Logic sequence

- The version of the move is determined. It is the delayed projectile version if `delprojs` is empty and a 33% RNG check passes (41% instead if hardmode is true) and the multi hits version otherwise
- animstate set to 101
- Yield for 0.5 seconds
- `Charge7` sound plays
- animstate set to 102
- A `Prefabs/Particles/Charging` particles is instantiated rooted and positioned at this enemy position + (1.15, 2.8, -0.1)

#### Multi hits version
If using the multi hits version of the move:

- The amount of hits is determined. It's always 2 (random between 2 and 3 instead if hardmode is true). A new array of GameObject is created to hold each projectiles with the amount being the length
- For each hit:
    - animstate set to 102
    - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable to get a target player party member. If the result isn't -1:
        - Yield for 0.65 seconds (0.5 seconds instead if hardmode is true)
        - The projectile element is set to a new `Prefabs/Objects/EnergySphere` GameObject rooted positioned at this enemy + (1.15, 2.8, -0.1) with `NoMapColor` tag
        - `Lazer2` sound plays
        - Projectile 1 call happens
        - `Prefabs/Particles/Charging` particles stops playing
    - animstate set to 101
    - If this isn't the last hit:
        - Yield for 0.5 seconds
        - `Prefabs/Particles/Charging` particles resumes playing
- Yield all frames until all projectile elements are null (meaning all Projectile calls completed)

#### Delayed projectile version
If using the delayed projectile version of the move:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- A new `Prefabs/Objects/EnergySphere` GameObject is created rooted positioned at this enemy position + (1.15, 2.8, -0.1) with `NoMapColor` tag
- `Lazer2` sound plays
- `Prefabs/Particles/Charging` particles stops playing
- animstate set to 101
- Over the course of 51.0 frames, `Prefabs/Objects/EnergySphere` moves to (0.0, 20.0, 0.0) via a lerp
- AddDelayedProjectile 1 call happens
- If hardmode is true, `cantmove` is decremented which gives an additional actor turn to this enemy

#### Cleanup
This happens no matter which version of the move used:

- `Prefabs/Particles/Charging` moved offscreen at -9999.0 in y then gets destroyed in 5.0 seconds

## Move 3 - Enemy summon
Summons a new [Midge](Midge.md) enemy. No damages are dealt, but the enemy gains a free actor turn if hardmode is true where damages may be dealt.

### Logic sequence

- A Vector3 is attempted to be generated that will corresponds to the position to spawn the new enemy. Each attempt may fail, but up to 20 attempts will be done and the first one that fufills some conditions will be used. Each attempts generates a vector where the x component is between 0.25 and 7.0, the y component is 0.0 and the z component is between -0.75 and 0.75. For the location to be accepted, any enemy part member needs to be located at a distance less than its `size` * 1.75. If no suitable locations is found after 20 attempts, the entire action logic restarts (including the pre move logic)
- ShakeScreen called with 0.1 ammount and 1.0 time with dontreset
- animstate set to 103
- `Growl` sound plays with 1.2 pitch
- Yield for 0.5 seconds
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new [Midge](Midge.md) with type `Offscreen` at the location without cantmove (meaning the new enemy can act immediately on the same main turn)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- The new enemy's `exp` is set to 0
- animstate set to 0 (`Idle`)
- [RegorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called
- If hardmode is true, `cantmove` is decremented which gives an additional actor turn to this enemy
