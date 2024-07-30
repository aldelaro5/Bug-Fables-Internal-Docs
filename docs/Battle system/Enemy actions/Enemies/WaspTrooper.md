# `WaspTrooper`

## MainManager.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy features special exp handling logic. Check the documentation to learn more.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the spear thrust move, the [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call is done with a multiplier of 2.7 instead of 2.5
- In the spear throw move, the yield interval before the throw is random between 0.6 and 0.9 seconds instead of being between 0.6 and 0.75 seconds
- In the spear throw move, the amount of frames that the spear takes to reach its target is random between 37.0 and 49.0 inclusive instead of always being 60.0

## Move selection
3 moves are possible:

1. A single target spear thrust
2. A single target spear slash
3. A single target spear throw

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/8|
|2|2/8|
|3|3/8|

## Move 1 - Spear thrust
A single target spear thrust

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 23 (`Chase`)
- Done 3 times:
    - `Jump` sound played
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - Yield for 0.25 seconds
    - Yield all frames while `onground is true
- Camera moves to look near `playertargetentity`
- animstate set to 23 (`Chase`)
- Yield a frame
- `trail` set to true
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.25, 0.0, -0.1) with 2.5 multiplier (2.7 instead if hardmode is true) using 23 (`Chase`) as walkstate and 100 as stopstate
- Yield all frames until `forcemove` is done
- `trail` set to false
- `Woosh4` sound plays
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 2 - Spear slash
A single target spear slash

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Woosh5` sound plays
- animstate set to 101
- Yield for 0.5 seconds
- Camera moves to look near `playertargetentity`
- animstate set to 23 (`Chase`)
- Yield a frame
- `trail` set to true
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with 2.25 multiplier using 23 (`Chase`) as walkstate and 102 as stopstate
- Yield all frames until `forcemove` is done
- `Woosh` sound plays
- `trail` set to false
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 3 - Spear throw
A single target spear throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 103
- Yield for a random interval between 0.6 and 0.75 seconds (the upper bound is 0.9 seconds instead if hardmode is true)
- animstate set to 28 (`TossItem`)
- `Toss3` sound plays
- `Toss4` sound plays
- A new sprite object is created rooted using `projectilepsrites[6]` sprite (a spear) at this enemy position + (0.0, 1.5, -0.1)
- Over the course of 60.0 frames (random between 37.0 and 50.0 frames instead if hardmode is true), the spear moves to `playertargetentity` position + (0.75, 1.25, -0.1) via a BeizierCurve3 with a ymax of 5.0 and its z angle changes from 50.0 to 140.0 via a lerp
- The spear gets destroyed
- DoDamage 1 call happens
- Yield for 0.5 seconds
- `Woosh5` sound plays
- animstate set to 101
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)
