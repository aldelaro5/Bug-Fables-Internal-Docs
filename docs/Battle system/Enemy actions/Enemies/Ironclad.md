# `Ironclad`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the projectile beam, the projectile takes 1 / 0.04 frames (25 frames) for the projectile to move to its target instead of 1 / 0.03 frames (~33.3333333334 frames)
- In the dash attack move, the amount of frames this enemy takes to reach its target changes to 13.0 frames from 20.0 frames

## Move selection
3 moves are possible:

1. A multiple targets projectile beams
2. Prepares for a bigger attach on their next actor turn by setting their `charge` to 2 and gaining the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
3. A single target dash attack

Move 3 is always (and only used) if `basestate` isn't 0 (`Idle`) meaning move 2 was used on their last actor turn as it sets the `basestate` away from 0.

As for the other 2 moves, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

## Move 1 - Projectile beam
A multiple targets projectile beams.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but each calls requires that at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3|[Sleep](../../Damage%20pipeline/AttackProperty.md#attackproperty)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new GameObject rooted with a SpriteRenderer using the `projectilepsrites[4]` sprite (a bolt projectile)|0.03 (~33.33333334 frames of movement) if hardmode is false, 0.04 if it's true (25 frames of movement)|0.0|`keepcolor`|`deathsmokelow`|null|null|Vector3.zero|false|

### Logic sequence

- `checkingdead` is set to new coroutine called SnailBeam which will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` is null

This is what the coroutine effectively does:

- Done 2 times:
    - If there's at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
        - Camera moves to look near the midpoint between this enemy and `playertargetentity` position
        - animstate set to 100
        - Yield for 0.5 seconds
        - animstate set to 101
        - `Lazer` sound plays
        - The projectile GameObject is created rooted with position being this enemy position + (-1.5, 1.5, -0.1), angles of 90.0 in z, scale of (0.5, 0.5, 0.5), sprite of `projectilepsrites[4]` sprite (a bolt projectile) and magenta color (the color is ignored because of the `keepcolor` argument sent to Projectile)
        - Projectile call 1 happens
        - Yield a random interval between 0.45 and 0.6 seconds
- Yield all frames until the projectile is null (it was destroyed meaning Projectile is done)
- Yield for 0.5 seconds
- `checkingdead` set to null informing the caller that the coroutine completed

## Move 2 - Prepares a bigger attack with 2 `charge` and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md)
Prepares for a bigger attach on their next actor turn by setting their `charge` to 2 and gaining the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Camera moves to look near this enemy
- `Charge7` sound plays
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- animstate set to 102
- Yield for 0.5 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enmey with type 4 (green up arrow)
- `StatUp` sound plays
- Yield for 0.5 seconds
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enmey with type 1 (blue up arrow)
- Yield for 0.5 seconds
- `basestate` and the local startstate are set to 103 which allows this enemy to use the dash attack on their next actor turn
- `charge` set to 2
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy for 2 main turns (effectively 1 main turn since the current main turn advances soon enough)

## Move 3 - Dash attack
A single target dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity` position
- ShakeSprite called with 0.1 intensity and 45.0 frametimer
- MainManager.MoveTowards + 1.0 in x with 30.0 frametime with smooth without local (done in paralel)
- `Charge7` sound plays
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 23 (`Chase`)
- `MothflyATK` sound plays
- `trail` set to true
- Over the course of 20.0 frames (13.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` + (1.0, 0.0, -0.1) via a lerp
- ShakeScreen called with 0.15 ammount and 0.5 time with dontreset
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE):
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `playertargetentity`
    - SlowSpinStop called on `playertargetentity` with a spinammount of 50.0 in y and 30.0 frametime
    - Over the course of 21.0 frames, this enemy moves to (-15.0, 0.0, 0.0) via a lerp
    - Over the course of 41.0 frames, this enemy moves from (15.0, 0.0, 0.0) to startp via a lerp
- Otherwise (blocked, ignores FRAMEONE):
    - animstate set to 1 (`Walk`)
    - Over the course of 51.0 frames, this enemy moves to its position + (2.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 3.0
- `trail` set to false
- The local startstate and `basestate` set to 0 (`Idle`) which prevents this move to be used on the next actor turn
