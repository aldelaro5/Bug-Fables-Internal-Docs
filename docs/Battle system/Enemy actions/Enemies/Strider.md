# `Strider`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the dash attack move, the amount of frames this enemy takes to move changes to 41.0 frames from 49.0 frames
- In the water projectiles throw move, the amount of hits changes to be random from 2 to 3 inclusive instead of always being 2
- In the water projectiles throw move, the speed of each Projectile call changes to 25.0 from 32.0

## Move selection
2 moves are possible:

1. A single target dash attack
2. A multiple targets water projectiles throw

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/2|
|2|1/2|

## Pre move logic
The following logic is always done before using a move if [position](../../Actors%20states/BattlePosition.md) is `Flying`:

- `overrideanimfunc` set to true
- ExtraAnimPlay called with name `EndFly`
- `Rope1` sound plays with 0.8 pitch
- Over the course of 26.0 frames, `height` changes to 0.0 via a lerp
- `overrideanimfunc` set to false 

## Post move logic
The following logic is always done after a move is used:

- If a 5/10 RNG check passes:
    - Yield for 0.5 seconds
    - `overrideanimfunc` set to true
    - `Rope1` sound plays
    - ExtraAnimPlay called with name `StartFly`
    - Over the course of 26.0 frames, `height` changes from 0.0 to 1.8 via a lerp
    - [position](../../Actors%20states/BattlePosition.md) set to `Flying`
    - `overrideanimfunc` set to false 
- Otherwise (5/10 chance to fail the RNG check):
    - [position](../../Actors%20states/BattlePosition.md) set to `Ground`

## Move 1 - Dash attack
A single target dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 23 (`Chase`)
- Over the course of 21.0 frames, this enemy moves to + 2.0 in x via a lerp
- ShakeSprite called with an intensity of 0.1 and 30.0 frametimer
- Yield for 0.5 seconds
- Camera moves to look near `playerdata[playertargetID]`
- `Toss15` sound plays
- Over the course of 49.0 frames (41.0 frames instead if hardmode is true), this enemy moves to (-15.0, 0.0, `playertargetentity` z position - 0.1) via a lerp. On the first game this enemy x position becomes lower than `playertargetentity` x position + 1.0:
    - DoDamage 1 call happens
    - If `commandsuccess` is true (blocked, ignores FRAMEONE), the movement stops abruptly
- If `commandsuccess` is true (blocked, ignores FRAMEONE):
    - `Toss14` sound plays
    - animstate set to 0 (`Idle`)
    - Over the course of 36.0 frames, this enemy moves to `playertargetentity` position + 2.0 in x via a BeizierCurve3 with a ymax of 3.0
    - Yield for 0.25 seconds
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with a 2.0 multiplier
    - Yield all frames until `forcemove` is done
    - `flip` set to false
- Otherwise (didn't block, ignores FRAMEONE):
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - Over the course of 26.0 frames, this enemy moves from (15.0, 0.0, 0.0) to startp via a SmoothLerp
    - animstate set to 0 (`Idle`)

## Move 1 - Water projectiles throw
A multiple targets water projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times (random between 2 and 3 times instead if hardmode is true), but the call and further calls won't happen if there's not at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|null|This enemy|`playertargetID`|A new sprite objected named `temp` rooted using the `projectilepsrites[15]` sprite (an inverted honey projectile so it looks like a water projectile) positioned at this enemy + (-0.5, 1.4, -0.1) with a scale of (0.65, 0.4, 1.0)|32.0 (25.0 instead if hardmode is true)|0.0|null|`WaterSplash`|`WaterSplash`|null|Vector3.zero|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called (this call won't do anything here as it will get recalled later)
- Camera moves to look neat the midpoint between this enemy and `partymiddle`
- The amount of hits is determined. It's always 2 (random between 2 and 3 instead if hardmode is true) and a SpriteRenderer array is initialised with this amount as the length to hold the projectiles
- For each hits to do:
    - If there's at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
        - animstate set to 100
        - `Charge19` sound plays with 1.1 pitch
        - Yield for 0.5 seconds
        - `WaterOut` sound plays
        - animstate set to 101
        - The projectile element is set to a new sprite objected named `temp` rooted using the `projectilepsrites[15]` sprite (an inverted honey projectile so it looks like a water projectile) positioned at this enemy + (-0.5, 1.4, -0.1) with a scale of (0.65, 0.4, 1.0)
        - Projectile 1 call happens
        - Yield for 0.25 seconds
        - If this is the last hit, animstate is set to 0 (`Idle`)
        - Yield for a random interval between 0.35 and 0.65 seconds
    - Otherwise:
        - animstate set to 0 (`Idle`)
        - No more hits are performed
- Yield all frames until all projectiles objects are null (all Projectile 1 calls completed)
