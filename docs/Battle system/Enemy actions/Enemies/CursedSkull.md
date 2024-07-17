# `CursedSkull`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the aerial strike move, the amount of frames this enemy takes to reach the target is always 21.0 frames instead of being 31.0 frames if neither of the following were true (it would still be 21.0 if either were true even if hardmode is false):
    - `forcefire` is true
    - The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`
- In the laser attack move, the yield time while charging the laser changes to 0.75 seconds from 0.85 seconds
- In the charging move, this enemy gets a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 2 main turns

## Move selection
3 moves are possible:

1. A single target aerial strike
2. A single target laser attack
3. Brace themselves by increasing their `charge` to explode on the next actor turn
4. A party wide explosition attack that self sacrifice

The decision of which moves to use is based on odds and they change depending on if [HPPercent](../../Actors%20states/HPPercent.md) is at least 0.9 or not. Here are the odds (move 3 and 4 are treated the same in this selection process):

|Move|Odds when [HPPercent](../../Actors%20states/HPPercent.md) >= 0.9|Odds when [HPPercent](../../Actors%20states/HPPercent.md) < 0.9|
|---:|-------|------|
|1|3/8|3/10|
|2|3/8|3/10|
|3-4|2/8|4/10|

There are however 2 special cases:

- If at least one of the following is true, move 3-4 is always used:
    - `basestate` isn't 0 (`Idle`)
    - `charge` is higher than 0
    - The current [map](../../../Enums%20and%20IDs/Maps.md) is the `TestRoom`
- If move 3-4 is selected, but [HPPercent](../../Actors%20states/HPPercent.md) is higher than 0.8, move 1 will be used instead

As for move 3-4, if it is selected and [HPPercent](../../Actors%20states/HPPercent.md) is 0.8 or lower the decision on which move to use between the 2 depends on `locktri`:

- If it is false, move 3 is used 
- If it is true, move 4 is used

> NOTE: Since it is possible to clear both the `charge` of this enemy as well as having its `basestate` set away from its charge one due to incorrect [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)'s falling logic, it is possible to have the move selection manipulated in such a way that move 4 won't be used on the next actor turn after move 3. In such case, they can still explode, but it has to happen through the move selection process as if they were charging.

## Pre move logic
The following logic always happen at the start of the action:

- If [position](../../Actors%20states/BattlePosition.md) is `Ground`, it is changed to `Flying` with the following:
    - `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call to change this enemy position to `Flying`
    - Yield all frames until `checkingdead` is null (the coroutine completed)

## Move 1 - Aerial strike
A single target aerial strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Happens if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|This enemy|The selected `playertargetID`|3|[Fire](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|3|Happens if DoDamage 1 didn't|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (3.0, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- `Charge10` sound plays
- ShakeSprite called with intensity of (0.1, 0.0, 0.0) and frametimer of 40.0
- Yield for 0.66 seconds
- The amount of frames for the dash is determined. It's 31.0 frames unless any of the following is true where it's 21.0 frames:
    - hardmode is true
    - `forcefire` is true
    - The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`
- Over the course of the amount of frames determined, this enemy moves to `playertargetentity` position + (-3.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 0.0 - `height`. On the first frame this enemy x position becomes lower than `playertargetentity` x position, DoDamage 1 or 2 call happens depending on the one whose conditions are met

## Move 2 - Laser attack
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

- If this enemy x position is less than 2.0:
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (2.5, 0.0, -0.5) with a 2.0 multiplier
    - Yield all frames until `forcemove` is done
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 100
- `Charge11` sound plays with 1.1 pitch
- A new `Prefabs/Particles/Charging` GameObject is created rooted positioned at `sprite` position + (0.0, 1.0, -0.1)
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

## Move 3 - Prepare for explosion
Brace themselves by increasing their `charge` to explode on the next actor turn. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- y `spin` set to 20.0
- `Charge13` sound plays
- animstate set to 100
- Yield for 0.75 seconds
- animstate set to 101
- `spin` zeroed out
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- If hardmode is true, [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy for 2 main turns without effect
- `StatUp` sound plays
- `charge` set to 2
- The local startstate and `basestate` set to 101
- `locktri` is set to true so the explosion move can be used on the next actor turn
- Yield for 0.5 seconds

## Move 4 - Explosion
A party wide explosition attack that self sacrifice.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Happens if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|This enemy|2|[Fire](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|
|2|Happens if DoDamage 1 didn't and `inice` is true|This enemy|1|[Freeze](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|
|3|Happens if DoDamage 1 and 2 didn't|This enemy|1|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- Camera moves to look near (-4.0, 1.0, 0.0)
- `bobspeed` and `bobrange` set to 0.0
- `Charge16` sound plays
- Over the course of 151.0 frames, this enemy moves to `partymiddle` via a SmoothLerp
- `Explosion5` sound plays
- ShakeScreen called with 0.3 ammount and 0.75 time
- DoDamage 1, 2 or 3 call happens depending on which one has their conditions met, but depending on which one happens, particles plays at `partymiddle` with a scale of 2.0x:
    - 1: `Fire`
    - 2: `mothicenormal`
    - 3: `explosionsmall`
- `selfsacrifice` set to true which prevents some logic to happen in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) since this enemy is set to die soon
- [CleanKill](../../Actors%20states/CleanKill.md) called on this enemy which turns this enemy into a blank actor shell placing their battleentity at startp
- Yield for 0.5 seconds
