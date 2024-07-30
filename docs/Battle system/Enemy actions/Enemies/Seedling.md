# `Seedling`

## [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) special logic
Before this enemy is loaded, it's possible that StartBattle overrides it to a `GoldenSeedling`. This also applies to the `FlyingSeedling` variant. See the documentation to learn more.

## Move selection
2 move are possible:

1. A single target aerial strike attack
2. A single target tackle attack

Move 1 is always done when the [position](../../Actors%20states/BattlePosition.md) is `Flying` while move 2 is always done otherwise.

## Move 1 - Aerial strike attack
A single target aerial strike attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- `Toss2` sound plays via PlayMoveSound on loop using `sounds[9]` with pitch 1.1 and volume 0.5
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (2.0, 0.0, -0.15) with 1.5 multiplier and stopstate 100
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `height` set to 0.0
- y position set to the `height` value before this action
- animstate set to 26 (`AirTackle`)
- `FastWoosh` sound plays
- Over the course of 1 / 0.055 frames (~18.181818 frames), this enemy moves to its current position + 0.75 in x via a BeizierCurve3 with a ymax of -0.01
- animstate set to 100
- Yield for 0.2 seconds
- animstate set to 26 (`AirTackle`)
- `Turn2` sound plays
- Over the course of 1 / 0.07 frames (~14.285714 frames), this enemy moves to `playerdata[playertargetID]` position + (0.0, 1.0, -0.1) via a lerp
- DoDamage call 1 happens
- Yield for 0.15 seconds
- `height` set to 0.2
- animstate set to 1 (`Walk`)
- Over the course of 1 / 0.045 frames (~22.22222 frames), this enemy moves to its position before the BeizierCurve3 movement earlier via a lerp
- y position is set to 0.0
- `height` is restored to its value before this action
- Yield for 0.2 seconds
- `bobrange` reset to `startbf`
- `bobspeed` reset to `startbs`
- FlipAngle called with setangle

## Move 2 - Tackle attack
A single target tackle attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to new coroutine called SeedlingTackle which will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` is null
- FlipAngle called with setangle

This is what the coroutine effectively does:

- animstate set to 23 (`Chase`)
- Done 2 times:
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - `Jump` sound plays with 0.75 pitch and 1.6 volume
    - Yield for 0.1 seconds
    - Yield all frames until `onground` goes to true
    - Yield a frame
- [CameraFocusTarget](../../Visual%20rendering/CameraFocusTarget.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + 3.0 in x with a 2.0 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- Yield for 0.2 seconds
- `Spin10` sound plays
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + (0.35, 0.0, -0.1) with a 2.5 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- DoDamage 1 call happens
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- `flip` set to false (with `overrideflip`)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + 2.0 in x with a 0.75 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- `flip` is set to false
- `checkingdead` set to null indicating the caller that the coroutine completed
