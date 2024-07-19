# `MothflyCluster`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the aerial strike move, the amount of frames this enemy takes to reach its target changes to 11.0 frames from 17.0 frames

## Move selection
2 moves are possible:

1. A single target aerial strike
2. Summons 2 new [Mothfly](Mothfly.md) enemies

Move 2 is always (and only) used when this enemy is the last `enemydata` remaining.

If move 2 wasn't used, move 1 is always used.

## Pre move logic
The following logic is always done at the start of the actor turn if [position](../../Actors%20states/BattlePosition.md) is `Ground`:

- `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to set the [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` set to null when completed)
- Yield all frames until `checkingdead` is null

## Move 1 - Aerial strike
A single target aerial strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (3.0, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 11 (`Hurt`)
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- `Charge19` sound plays
- Yield for 0.5 seconds
- animstate set to 100
- `Toss13` sound plays
- Over the course of 17.0 frames (11.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` + (0.5, 0.0 - `height` + 0.5, -0.1) via a lerp
- ShakeScreen called with 0.075 ammount and 0.5 time with dontreset
- DoDamage 1 call happens
- Yield for 0.25 seconds
- Over the course of 21.0 frames, this enemy moves to `playerdata[playertargetID]` + (2.0, 0.0, -0.1) via a lerp

## Move 2 - Summons 2 [Mothfly](Mothfly.md)
Summons 2 new [Mothfly](Mothfly.md) enemies. No damages are dealt.

### Logic sequence

- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (4.0, 0.0, 0.0) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Toss11` sound plays
- startp set to this enemy position except for the y component which is set to 0.0
- `checkingdead` set to a new SummonMothFly coroutine (`checkingdead` is set to null when it completes)
- Yield all frames until `checkingdead` is null

This is what the SummonMothfly coroutine ends up doing:

- Camera moves to look near this enemy, but zoomed in
- animstate set to 11 (`Hurt`)
- `rotater` gets a SpriteBounce added with 0.0 `frequency` and 20.0 `speed`
- Over the course of 31.0 frames, the `rotater`'s SpriteBounce's `frequency` changes to 0.1 via a lerp
- Yield for 0.5 seconds
- The `rotater`'s SpriteBounce gets destroyed
- `rotater` scale increased by 30%
- `PingDown` sound plays
- Done 2 times:
    - [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to a new [Mothfly](Mothfly.md) enemy at this enemy + (0.0, `height`, 0.1)
    - Yield a frame
    - The `enemydata[lastaddedid]` gets some adjustements:
        - `maxhp` and `hp`: divided by 2 floored
        - `exp`: 0
        - `money`: 0
        - y `spin`: 20.0
- Over the course of 31.0 frames, `rotater` scale changes to its original value before this coroutine via a lerp and the 2 new enemies that were just summoned moves from this enemy position to (`x`, 0.0, -0.1) via a BeizierCurve3 with a ymax of 4.0 where `x` is 1.0 for the first enemy and 7.0 for the second enemy
- `rotater` scale is set to its value before this coroutine
- Both enemies who were just summoned has their `spin` zeroed out
- `checkingdead` is set to null which signals the caller that this coroutine completed
