# `Krawler`

## [StartBattle](../../StartBattle.md) special logic
This enemy has special logic that happens on StartBattle after it is loaded:

- If calledfrom exists and its entity is `inice` or the battleentity is:
    - `freezeres` increased by 70
    - `inice` set to true
    - [weakness](../../Actors%20states/Enemy%20features.md#weakness) set to a new list with one element being [HornExtraDamage](../../Damage%20pipeline/AttackProperty.md)

## [Fire](../../Actors%20states/BattleCondition/Fire.md) damage infliction logic in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This enemy has a 50% chance to be on fire on a `Fire` property attack if the current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`. It cannot be on fire otherwise.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the laser attack move, the yield time while charging the laser changes to 0.75 seconds from 0.85 seconds
- In the dash attack move, the amount of frames this enemy takes to reach the target is always 21.0 frames instead of being 27.0 frames if neither of the following were true (it would still be 21.0 if either were true even if hardmode is false):
    - `forcefire` is true
    - The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`

## Move selection
2 moves are possible:

1. A single target laser attack
2. A single target dash attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|40%|
|2|60%|

## Move 1 - Laser attack
A single target laser attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Happens if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|This enemy|The selected `playertargetID`|3|[Fire](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|2|Happens if DoDamage 1 didn't and `inice` is true|This enemy|The selected `playertargetID`|2|[Freeze](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|3|Happens if DoDamage 1 and 2 didn't|This enemy|The selected `playertargetID`|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- If this enemy x position is less than 2.0:
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (2.5, 0.0, -0.5) with a 2.0 multiplier
    - Yield all frames until `forcemove` is done
- animstate set to 100
- Yield for 0.5 seconds
- `Charge11` sound plays with 1.1 pitch
- A new `Prefabs/Particles/Charging` GameObject is created rooted positioned at `sprite` position + (0.0, 2.0, -0.1)
- Yield for 0.85 seconds (0.75 seconds instead if hardmode is true)
- `Shot` sound plays
- `Prefabs/Particles/Charging` moved offscreen at 999.0 in y
- `Prefabs/Particles/Charging` gets destroyed in 5.0 seconds
- The color of the laser is determined:
    - It's pure red if all at least one of the following is true:
        - `forcefire` is true
        - The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`
    - Otherwise, it's pure cyan if `inice` is true
    - Otherwise, it's pure yellow
- A new LightingBolt coroutine starts which will use a `Prefabs/Particles/LightingBolt` to animate a lighting using its TrailRenderer. The coroutine only contains visual logic that doesn't affect the timing of the block so its logic will be omitted
- Yield for 0.1 seconds
- DoDamage 1, 2 or 3 happens depending on which one fits its condition requirements
- Yield for 0.5 seconds

## Move 2 - Dash attack
A single target dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Happens if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|This enemy|The selected `playertargetID`|4|[Fire](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|2|Happens if DoDamage 1 didn't|This enemy|The selected `playertargetID`|3|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Scuttle3` sound plays on loop with 1.2 pitch
- Camera moves to look near the midpoint between this enemey and `playertargetentity`
- animstate set to 1 (`Walk`)
- `Prefabs/Particles/WalkDust` particles are instantiated childed to the `sprite` with a local position (0.0, 0.2, -0.1) with its EmissionModule's rateOverTime set to 20.0
- ShakeSprite called with an intensity of (0.1, 0.1, 0.1) and frametimer of 50.0
- Yield for a random time between 0.75 and 1.0 seconds
- `Shot2` sound plays
- Camera moved to look near `playertargetentity` zoomed
- `Prefabs/Particles/WalkDust`'s MainModule's startLifetime set to a constant curve with a value of 1.5 and its startSize set to 1.25
- The amount of frames for the dash is determined. It's 27.0 frames unless any of the following is true where it's 21.0 frames:
    - hardmode is true
    - `forcefire` is true
    - The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`
- Over the course of the amount of frames determined, this enemy moves to `playertargetentity` position + (0.4, 0.0, -0.1) via a lerp
- `Scuttle3` sound stops
- ShakeScreen called with an ammount of 0.3 and time of 0.65
- `Death3` sound plays
- DoDamage 1 or 2 call happens depending on which one has its conditions requirements fufilled
- `Prefabs/Particles/WalkDust` moves offscreen at 999.0 in y then destroyed in 2.0 seconds
- Over the course of 41.0 frames, this enemy moves to + 2.25 in x via a BeizierCurve3 with a ymax of 2.25
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
