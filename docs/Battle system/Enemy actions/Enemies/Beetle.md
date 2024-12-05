# `Beetle`

> NOTE: This page is about the enemy action, NOT the player party member.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to taunt before using a move changes to 30% from 20%
- In the rock throw move, the amount of frames the rock takes to move to the target is 41 frames instead of 48 frames
- In the dash move, the amount of frames the entire dash takes is 25 instead of 33

### `dontusecharge` might be set to true
This move may sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

It is only done when the taunting pre move logic is performed which has a random chance to occur (20% or 30% if hardmode is true). Due to the randomness of this, it is very likely that this logic is incorrect because it means that if [Kali](Kali.md) were to set their `charge` to 2, there is a random chance that the move performed will not consume any charges no matter what the move is which isn't supposed to happen.

## Move selection
4 moves are possible:

1. A single target horn slash
2. A single target underground strike
3. A single target rock throw
4. A party wide dash attack

Here are the odds of using each move:

|Move|Odds|
|---:|----|
|1|2/7|
|2|2/7|
|3|2/7|
|4|1/7|

## Pre move logic
Before using any moves, if a 20% RNG check passes (30% instead if hardmode is true), this enemy will perform a move that inflicts the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on a target. This isn't a dedicated move because if it is performed, the move selection still proceeds on the same actor turn after that so it's a still a pre move logic

Here is the taunt move logic performed if the RNG check passes (done by yield returning the EnemyTaunt coroutine with the entity):

- `dontusecharge` is set to true. NOTE: This is incorrect, see the section above explaining it
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 109
- Camera moves to look near this enemy
- `Taunt2` sound plays on loop using `sounds[9]`
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on `playerdata[playertargetID]` for 2 main turns (effectively 1 since the main turn advances soon after)
- `Taunt` sound plays
- animstate of `playertargetentity` set depending on `playertargetID` (their `trueid`)
    - 0 (`Bee`): 10 (`Flustered`)
    - 1 (`Beetle`): 5 (`Angry`)
    - 2 (`Moth`): 102
- `Prefabs/Particles/AngryParticle` instantiated childed to `playertargetentity` with local position being Vector3.up
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `sounds[9]` stopped
- `Prefabs/Particles/AngryParticle` destroyed
- `playertargetentity` animstate restored to its value before this action

## Move 1 - Horn slash
A single target horn slash

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence
This is done by yield returning the EnemyHeavyStrike coroutine with the entity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) `playertargetentity` position + (1.5, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 103
- `Charge3` sound plays using `sounds[9]` with 1.2 pitch
- ShakeSprite called with intensity (0.25, 0.0, 0.0) and 45.0 frametimer
- Yield for 0.75 seconds
- `sounds[9]` stopped
- animstate set to 104
- `HugeHit` sound plays
- ShakeScreen called with 0.2 ammount for 0.75 time with dontreset
- DoDamage 1 call happens
- `playertargetentity`'s y `spin` set to 35.0
- `playertargetentity` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0
- Yield for 0.5 seconds
- Yield all frames until `playertargetentity` is `onground`
- `playertargetentity`'s `spin` zeroed out
- Yield for 0.5 seconds

## Move 2 - Underground strike
A single target underground strike

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence
This is done by yield returning the EnemyKabbuDig coroutine with the entity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Dig` sound plays
- animstate set to 101
- `digging` set to true
- Yield for 0.65 seconds
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) `playertargetentity` position with 2.0 multiplier
- `Digging` sound plays on loop using `sounds[9]`
- Yield all frames until `forcemove` is done
- Yield for 0.25 seconds
- ShakeScreen called with 0.1 ammount for 0.75 time with dontreset
- `sounds[9]` stopped
- DoDamage 1 call happens
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `digging` set to false
- `sprite` angles reset to Vector3.zero
- y `spin` set to 20.0
- animstate set to 119
- `sprite` scale set to `startscale`
- `DigPop` sound plays
- `DirtExplode` particles plays at `playertargetentity` position
- Over the course of 46.0 frames, this enemy moves to the value before this coroutine + Vector3.up via a BeizierCurve3 with a ymax of 0.0
- position set to the value before this coroutine
- animstate set to 13 (`BattleIdle`)
- SetAnimForce called
- `spin` zeroed out

## Move 3 - Rock throw
A single target rock throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|null|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyPebbleToss coroutine with the entity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 28 (`TossItem`)
- `Toss` sound plays
- A new sprite object is created rooted at this enemy postion + (0.5, 2.0, -0.1) using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite
- Over the course of 48.0 frames (41.0 instead if hardmode is true), the `MightyPeeble` sprite moves to `playertargetentity` position + (0.0, 2.0, -0.1) via a BeizierCurve3 with a ymax of 3.5. Before each frame yield, `MightyPeeble`'s z angle increses by 15.0 * framestep
- DoDamage 1 call happens
- `MightyPeeble` object is destroyed
- Yield for 0.5 seconds

## Move 4 - Dash attack
A party wide dash attack

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|`commandsuccess`|0.0|Vector3.zero|false|null|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- `checkingdead` is set to a new BeetleDash coroutine with the following parameters (`checkingdead` is set to null when completed):
    - enemyid: This enemy
    - damage: 3
    - animprepare: 116
    - animdash: 117
    - startp: startp
    - speed: 33 (25 instead if hardmode is true)
- Yield all frames until `checkingdead` is null

This is what the coroutine effectively ends up doing:

- `playertargetID` set to -1
- Camera moves slightly to the left
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-0.5, 0.0, 0.0) at 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Charge3` sound plays using `sounds[9]` with 1.2 pitch
- animstate set to 116
- ShakeSprite called with (0.25, 0.0, 0.0) intensity and 60.0 frametimer
- Over the course of 41.0 frames, this enemy moves 2.0 to the left via a lerp
- ShakeSprite called with 0.1 intensity and 40.0 frametimer
- Yield for 41.0 frames counted by framestep
- `sounds[9]` stopped
- animstate set to 117
- `trail` set to true
- `FastWoosh` sound plays
- Over the course of 34 frames (26 frames instead if hardmode is true), this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame that the x position is less than -4.5, the following happens only once:
    - ShakeScreen called with an ammount of (0.25, 0.0, 0.0) for 0.65 time with dontreset
    - `HugeHit` sound plays
    - PartyDamage 1 call happens
- position set to (15.0, 0.0, 0.0)
- `trail` set to false
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 1 (`Walk`)
- Over the course of 61.0 frames, this enemy moves to startp via a lerp while having its animstate set to 1 (`Walk`)
- `sprite` local position reset to Vector3.zero
- `checkingdead` set to null which signals the caller that this coroutine completed
