# `CordycepsAnt`

## Move selection
1 move is possible:

1. A single target slashing attack

Move 1 is always done.

## Move 1 - slash attack
A single target slashing attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Scuttle` sound plays on `sounds[9]` at 0.8 pitch 0.7 volume on loop
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) slightly to the right of `playertargetID` player party member with a walkstate of 23 (`Chase`)
- Yield until `forcemove` is done
- `sounds[9]` stopped
- Yield for 0.1 seconds
- `Blosh` sound plays
- animstate set to 100
- Yield for 0.35 seconds
- animstate set to 101
- `Chew` sound plays
- Yield for 0.1 seconds
- DoDamage 1 happens
- Yield for 0.5 seconds
