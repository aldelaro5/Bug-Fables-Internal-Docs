# `Sandworm`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the underground attack move, the time the enemy takes to move towards its target by digging is 31.0 frames instead of 41.0 frames. This happens before the actual attack movement after undigging

## Move selection
1 moves are possible:

1. A single target underground attack

Move 1 is always used

## Pre move logic
The following logic always happen at the start of the actor turn:

- `checkingdead` is set to a new [Dig](../Dig.md) coroutine for this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Underground` with 0 jump (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null

## Move 1 - Underground attack
A single target underground attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Digging` sound plays on loop using `sounds[9]`
- Camera moves to look near the midpoint between this this enemy and `playertargetentity`
- Over the course of 51.0 frames, this enemy moves in x to startp.x + a BeizierFloat with a height of 2.25
- animstate set to 100
- Over the course of 41.0 frames (31.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1)
- `digging` set to false
- `spin` zeored out
- `digtime` set to 99.0
- FlipAngle called with setangle
- `sounds[9]` stopped
- `DigPop` sound plays
- Over the course of 21.0 frames, this enemy moves in x to `playertargetentity` x position + a lerp from 2.0 to -2.0 and `hright` is set to a BeizierFloat curve with a height of 1.6. On the first frame that this enemy x position is equal or less than `playertargetentity` x position, DoDamage 1 call happens
- Yield a frame
- `checkingdead` is set to a new [Dig](../Dig.md) coroutine for this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Underground` with 0 jump (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null
- animstate set to 0 (`Idle`)
