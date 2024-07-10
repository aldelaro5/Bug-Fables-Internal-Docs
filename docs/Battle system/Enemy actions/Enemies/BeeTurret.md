# `BeeTurret`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: The amount of actor turns left before being about to use the delayed projectile move again after it was used. This value is set to 4 when the delayed projectile move is used and in the pre move logic, it is decremented if it is higher than 0. This means normally, 4 actor turns needs to pass between each delayed projectiles usage.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the delayed projectile move, the amount of main turns needed for the projectile to land changes to 1 from 2
- In the honey projectiles move, the speed of each projectiles changes to 40.0 from 45.0
- In the rocket projectiles move, the speed of each projectiles changes to 30.0 from 35.0

## Move selection
3 moves are possible:

1. A single target honey projectiles throw that hits twice
2. A multiple targets rocket projectiles throw that hits twice
3. Adds a single target [Delayed projectile](../../Actors%20states/Delayed%20projectile.md)

The decision of which moves to use is based on the following odds:

|Move|Odds|
|1|2/5|
|2|2/5|
|3|1/5|

However, move 3 has the additional requirement that `data[0]` has to be 0 after the pre move logic for the move to be used. If it's selected, but this requirement isn't fufilled, Move 1 will be used instead, but with one change: the `Blosh` sound will be played instead of `Clomp` like it normally would.

## Pre move logic
The following logic always happen at the start of the actor turn:

- If `data[0]` is higher than 0, it is decremented

## Move 1 - Honey projectiles throw
A single target honey projectiles throw that hits twice.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but each call can only happen if at least 1 player party member is alive (`hp` more than 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new GameObject childed to `battlemap` with a SpriteRenderer using the `projectilepsrites[10]` sprite (a honey projectile) with the `spritemat` material positioned at this enemy position + (-1.0, -1.0, 0.1) on layer 14 (`Sprite`) with a SpriteBounce that has MessageBounce called on it with 3.0 multiplier|35.0 (30.0 instead if hardmode is true)|0.0|null|`ElecFast`|null|null|(0.0, 0.0, 20.0)|false|

### Logic sequence

- `Clomp` sound plays. NOTE: If this moved is used as a result from selecting move 2 when `data[0]` wasn't 0, `Blosh` sound plays instead
- animstate set to 100
- Yield for 0.95 seconds
- A new array of 2 Transform is created to hold the projectiles
- For each projectiles to throw:
    - If there is at least one alive player party member (`hp` above 0 and not [EatenBy](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
        - animstate set to 102
        - `Shot` sound plays
        - Projectile element initialized to a new GameObject childed to `battlemap` with a SpriteRenderer using the `projectilepsrites[10]` sprite (a honey projectile) with the `spritemat` material positioned at this enemy position + (-1.0, -1.0, 0.1) on layer 14 (`Sprite`) with a SpriteBounce that has MessageBounce called on it with 3.0 multiplier
        - Projectile 1 call happens
    - Yield for 0.15 seconds
    - If this isn't the last projectile:
        - animstate set to 100
        - Yield for 0.85 seconds
- Yield all frames until both projectiles are null (meaning all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 2 - Rocket projectiles throw
A multiple targets rocket projectiles throw that hits twice.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but each call can only happen if at least 1 player party member is alive (`hp` more than 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) and that [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1 (if the first call returns -1, the second one won't happen)|3|null|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|A new `Prefabs/Objects/BeeRocket` GameObject childed to `battlemap` positioned at this enemy position + (-1.25, 1.7, 0.1) with scale (0.5, 0.75, 1.0) on layer 14 (`Sprite`)|45.0 (40.0 instead if hardmode is true)|4.0|null|`explosionsmall`|`Explosion`|null|Vector3.zero|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- `Blosh` sound plays
- animstate set to 100
- Yield for 0.95 seconds
- A new array of 2 Transform is created to hold the projectiles
- For each projectiles to throw:
    - If there is at least one alive player party member (`hp` above 0 and not [EatenBy](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called. If it returns -1, this will be seen as the last projectile and this one will not be thrown (skips to the next Yield)
        - animstate set to 103
        - `b33BotInstantMissle` sound plays
        - Projectile element initialized to a new `Prefabs/Objects/BeeRocket` GameObject childed to `battlemap` positioned at this enemy position + (-1.25, 1.7, 0.1) with scale (0.5, 0.75, 1.0) on layer 14 (`Sprite`)
        - Projectile 1 call happens
    - Yield for 0.15 seconds
    - If this isn't the last projectile:
        - animstate set to 100
        - Yield for 0.85 seconds
- Yield all frames until both projectiles are null (meaning all Projectile 1 calls completed)
- Yield for 0.5 seconds


## Move 3 - Delayed projectile
Adds a single target [Delayed projectile](../../Actors%20states/Delayed%20projectile.md). No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect aything for this move.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Always happen|A new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at (0.0, 20.0, 0.0), a scale of (0.75, 1.0, 1.0) and a z angle of -75.0|`playertargetID`|3|2 (1 instead if hardmode is true)|0|null|70.0|This enemy|null|`explosion`|`@MisseleFall`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy, but a bit up
- animstate set to 100
- Yield for a second
- animstate set to 104
- `b33BotRocketLaunchInAir` sound plays
- A new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at this enemy position + (0.0, 2.0, 0.2), a scale of (0.75, 1.0, 1.0) and a z angle of -85.0
- HitPart particles plays at the `Prefabs/Objects/BeeRocket` position
- Over the course of 41.0 frames, the `Prefabs/Objects/BeeRocket` moves to (this enemy x position -1.0, 10.0, 0.0) via a SmoothLerp. After the first frame past 20.0 frames, animstate is set to 104
- The `Prefabs/Objects/BeeRocket` position is set to (0.0, 20.0, 0.0)
- The `Prefabs/Objects/BeeRocket` z angle is set to 75.0
- AddDelayedProjectile call happens
- `data[0]` set to 4 meaning this move won't be usable until 4 more actor turns passes on this enemy
