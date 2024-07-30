# `KeyR` and `KeyL`

> NOTE: Both `KeyR` and `KeyL`, while different enemies, they share the exact same action logic so they are documented together.

## [Fire](../../Actors%20states/BattleCondition/Fire.md) damage infliction logic in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This enemy cannot be on fire meaning any `Fire` property damages will not inflict the [Fire](../../Actors%20states/BattleCondition/Fire.md) condition.

## HPBarOnOther special logic
This enemy has special logic in a method called HPBarOnOther which is used by [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) to know if the `hpbar` should be shown despite this enemy not being spied yet. It returns true if [EverlastingKing](EverlastingKing.md) was spied which is needed because it's not possible under normal gameplay to spy this enemy, but it is possible to spy `EverlastingKing`. This means that this logic allows this enemy's `hpbar` to be displayed by spying `EverlastingKing`.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the aerial spinning strike move, the [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) call's frametime before the DoDamage call changes to 17.0 from 23.5

## Move selection
2 moves are possible:

1. A single target laser attack
2. A single target aerial spinning strike

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|49%|
|2|51%|

## Pre move logic
The following logic always happen before using a move if [position](../../Actors%20states/BattlePosition.md) is `Ground` to change it to `Flying`:

- Over the course of 31.0 frames, `height` changes from `minheight` to `initialheight` via a lerp
- [position](../../Actors%20states/BattlePosition.md) is set to `Flying`

## Move 1 - Laser attack
A single target laser attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `walkstate` set to 0 (`Idle`)
- `bobspeed` set to 0.0
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.1) with a 3.0 multiplier using 100 as the walkstate and stopstate
- Camera moves to look near the midpoint between `playertargetentity` and `forcetarget`
- Yield all frames until `forcemove` is done
- animstate set to 100
- A new `Prefabs/Particles/Charging` GameObject is instantiated childed to the `sprite` with a local position of -1.0 in y and a z angle of -45.0
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `Shot` sound plays
- `Prefabs/Particles/Charging`'s position set to offscreen at 999.0 in y then destroyed in 5.0 seconds
- `Lazer` particles plays at this enemy position with -1.0 alivetime then gets childed to the `sprite` with a local position of -1.0 in y and a x angle of 90.0
- DoDamage 1 call happens
- Over the course of 41.0 frames, the `Lazer` particles x and y scale changes to 0.0 via a lerp
- `Lazer` particles gets destroyed
- animstate set to 0 (`Idle`)
- `bobspeed` set to the value before this action

## Move 2 - Aerial spinning strike
A single target aerial spinning strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `walkstate` set to 0 (`Idle`)
- `bobspeed` set to 0.0
- `Spin` sound plays on loop using `sounds[9]`
- animstate set to 100
- `sprite` z angle set to -70.0
- y `spin` set to 30.0
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on this enemy to move them to + 1.5 in x with 30.0 framtime without changeanim
- Yield all frames until `forcemoving` is done
- Yield for 0.25 seconds
- Camera moves to look near `playerdata[playertargetID]`
- `trail` set to true
- `sounds[9]` stopped
- `Shot2` sound plays
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on this enemy to move them to `playertargetentity` position + (0.25, 1.0 - `height`, -0.1) with 23.5 framtime (17.0 instead if hardmode is true) without changeanim
- Yield all frames until `forcemoving` is done
- DoDamage 1 call happens
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `trail` set to false
- `sprite` angles reset to Vector3.zero
- `spin` set to (0.0, 0.0, 30.0)
- Over the course of 41.0 frames, this enemy moves to startp via a BeizierCurve3 with a ymax of 6.0
- `spin` zeroed out
- animstate set to 0 (`Idle`)
- `bobspeed` set to the value before this action
