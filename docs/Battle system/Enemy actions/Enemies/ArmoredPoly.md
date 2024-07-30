# `ArmoredPoly`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the rolling attack move, the multiplier of the [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call changes to 3.65 from of 3.0. This makes the enemy moves ~21.6666667% faster

## Move selection
1 move is possible:

1. A single target rolling attack

Move 1 is always done.

## Move 1 - Rolling attack
A single target rolling attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy position and `playerdata[playertargetID]`
- animstate set to 23 (`Chase`)
- `Curl` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.1 seconds
- Yield all frames until `onground` is true
- Yield for 0.3 seconds
- Camera moved to look near `playerdata[playertargetID]`
- `Spin2` sound plays on loop using `sounds[9]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + 0.5 in x - `globalcamdir`'s forward * 0.2 with a multiplier of 3.0 (3.65 if hardmode is true) and both the current animstate as the walkstate and stopstate
- Yield a frame
- Yield all frames until `forcemove` is done
- DoDamage 1 call happens
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- If `commandsuccess` is false (failed to block, ignores FRAMEONE):
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-20.0, 0.0, this enemy's z position) using the same multiplier as the call before and both the current animstate as the walkstate and stopstate
    - Yield all frames until `forcemove` is done. Before each frame yield, `playerdata[playertargetID]` has their animstate set to 11 (`Hurt`) and y `spin` to (0.0, 30.0, 0.0) * the distance between this enemy position and its `forcetarget` clamped from 0.0 to 1.0
    - `playerdata[playertargetID]` animstate set to 13 (`BattleIdle`)
    - `playerdata[playertargetID]`'s `spin` zeroed out
    - position set to (10.0, 0.0, startp.z)
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 as multiplier and both the current animstate as the walkstate and stopstate
    - Yield all frames until `forcemove` is done
- Otherwise:
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) current position + 1.5 in x with 1.5 as multiplier and both the current animstate as the walkstate and stopstate
- `sounds[9]` stopped
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.1 seconds
- Yield all frames until `onground` is true
- If `commandsuccess` is true (blocked, ignores FRAMEONE), `flip` is toggled
- animstate set to 0 (`Idle`)
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.4 seconds
