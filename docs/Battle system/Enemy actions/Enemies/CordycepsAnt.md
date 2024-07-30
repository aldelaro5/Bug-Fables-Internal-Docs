# `CordycepsAnt`

## Move selection
1 move is possible:

1. A single target slashing attack

Move 1 is always done.

## Move 1 - Slash attack
A single target slashing attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Scuttle` sound plays on loop using `sounds[9]` at 0.8 pitch and 0.7 volume on loop
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetID` position + (1.3, 0.0, -0.15) with 1.3333334 multiplier using 23 (`Chase`) as walkstate and 0 (`Idle`) as stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- Yield for 0.1 seconds
- `Blosh` sound plays
- animstate set to 100
- Yield for 0.35 seconds
- animstate set to 101
- `Chew` sound plays
- Yield for 0.1 seconds
- DoDamage 1 call happens
- Yield for 0.5 seconds
