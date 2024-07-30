# `ShockWorm`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- After attacking, the odds to set [isdefending](../../Actors%20states/Enemy%20features.md#isdefending) to true with 2 `defenseonhit` changes to 51% from 31% which also means that the chances they set `isdefending` to false with 0 `defenseonhit` changes to 49% from 69%

## Move selection
1 move is possible:

1. A single target electric shock attack

Move 1 is always used.

## Post move logic
The following logic is always done after using a move:

- If a 31% RNG check passes (51% instead if hardmode is true):
    - FlipAngle called with setangle
    - `TextBack` sound plays
    - [IsDefending](../../Actors%20states/Enemy%20features.md#isdefending) set to true
    - `defenseonhit` set to 2
    - animstate and the local startstate set to 24 (`Block`)
    - FlipSimple called
    - Yield for 0.5 seconds
- Otherwse (the RNG check failed):
    - If [isdefending](../../Actors%20states/Enemy%20features.md#isdefending) is true, `TextBack2` sound plays
    - The local startstate is set to 0 (`Idle`)
    - `isdefending` set to false
    - `defenseonhit` set to 0

## Move 1 - Electric shock attack
A single target electric shock attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.0, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- `Charge17` sound plays
- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 101
- DoDamage 1 call happens
- `ElecFast` particles plays at this enemy position + (0.0, 0.5, -0.1)
- `Shock` sound plays
- Yield for 0.7 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) startp with 2.0 multiplier
- Yield all frames until `forcemove` is done
