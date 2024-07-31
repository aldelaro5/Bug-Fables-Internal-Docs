# `LeafbugArcher`

## Assumptions
It is assumed this enemy is loaded with a non empty [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) as it is needed for their `hitaction` logic to work. Typically, it will be set to other `LeafbugArcher`, [LeafbugNinja](LeafbugNinja.md) and [LeafbugClubber](LeafbugClubber.md).

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to summon an enemy from the leafbug family when [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and this enemy is the last `enemydata` remaining increases to 5/10 from 3/10
- In the projectile throw move, the speed of the Projectile call changes to 20.0 from 27.0

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- If `charge` is less than 3, it is incremented with the following:
    - animstate set to 5 (`Angry`)
    - `Wam` sound plays
    - [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote with 20 time
    - Yield all frames until `emoticoncooldown` expires
    - `StatUp` sound plays
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
    - Yield for 0.5 seconds
    - `charge` is incremented

## Move selection
2 moves are possible:

1. A single target projectile throw
2. Adds a [delayed projectile](../../Actors%20states/Delayed%20projectile.md)

The decision of which move to use depends first on `charge`. If it is above 0, move 1 is always used.

Otherwise, the deicision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|5/8|
|2|3/8|

## Pre move logic
The following logic always happen at the start of the action if all of the following conditions are met to summon a new enemy from the leafbug family:

- This enemy is the last `enemydata` left
- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5
- A 3/10 RNG check passes (5/10 instead if hardmode is true)

Here is what the logic does if all the of above are fufilled:

- animstate set to 105
- Yield for 0.25 seconds
- `Horn` sound plays with 1.1 pitch and 0.7 volume
- Yield for 0.5 seconds
- `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon an enemy from the leafbug family with type `Offscreen` at a random location between (0.25, 0.0, -0.75) and (7.0, 0.0, 0.75) where the distance between any enemy party member is less than its `size` * 1.75 with cantmove (meaning the enemy needs to wait on the next main turn to act). As for the enemy to summon, it's random among the following:
    - [LeafbugNinja](LeafbugNinja.md)
    - Another `LeafbugArcher`
    - [LeafbugClubber](LeafbugClubber.md)
- Yield for 0.85 seconds
- animstate set to 0 (`Idle`)
- Yield all frames until `checkingdead` is null (the coroutine completed)

## Move 1 - Projectile throw
A single target projectile throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|A new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) positioned at this enemy + (-1.0, 1.2, -0.1) with a scale of 0.65x and a z angle of -90.0|27.0 (20.0 instead if hardmode is true)|0.0|null|null|null|null|Vector3.zero|false|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- A new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) with a scale of 0.65x positioned offscreen at -99.0 in y
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 100
- `Rope1` sound plays
- Yield for 0.75 seconds
- animstate set to 102
- `Toss6` sound plays
- The projectile element gets positioned at this enemy + (-1.0, 1.2, -0.1) with a z angle of -90.0
- Projectile 1 call happens
- Yield all frames until the projectile is null (Projectile 1 call completed)
- Yield for 0.5 seconds

## Move 2 - [Delayed projectile](../../Actors%20states/Delayed%20projectile.md)
Adds a [delayed projectile](../../Actors%20states/Delayed%20projectile.md). No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### nocharm set to true
This move always set the nocharm local to true which will prevent any [UseCharm](../../Battle%20flow/UseCharm.md) calls to occur in the post action phase. This means no `HealHP` charm can occur after this action.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Always happen|A new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) positioned at (0.0, 15.0, 0.0) with a scale of 0.65x and a z angle of -20.0|[GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) without nullable|3|1|0|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|50.0|This enemy|null|null|`@Toss4`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- A new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) with a scale of 0.65x positioned offscreen at -99.0 in y
- animstate set to 100
- `Rope1` sound plays
- Yield for 0.15 seconds
- animstate set to 103
- Yield for 0.5 seconds
- `Toss6` sound plays
- animstate set to 104
- The projectile has its z angle set to -150.0
- Over the course of 46.0 frames, the projectile moves from this enemy position + (-0.75, 2.15, -0.1) to (0.0, 15.0, 0.0) via a lerp
- The projectile has its z angle set to -20.0
- AddDelayedProjectile call 1 happens
