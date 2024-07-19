# `PrimalWeevil`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds for this enemy to get an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition in the pre move logic when it didn't had it already and the numbing scream or enemy summon wasn't used changes to be 41% from 31%
- In the grab attack move, the amount of frames this enemy takes to move before attempting to grab the target changes to 36.0 frames from 43.5 frames

## Move selection
5 moves are possible:

1. A party wide numbing scream attack
2. A single target swipe attack that may hit twice
3. A single target tail slash attack
4. A single target grab attack that may hit twice
5. Summons a new [Weevil](Weevil.md) enemy

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/8|
|2|2/8|
|3|2/8|
|4|1/8|
|5|2/8|

However, if move 5 is used while this isn't the only `enemydata` left, a continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection process.

## Pre move logic
The following logic is always done before using a move if all of the following conditions are fufilled to get an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition:

- The selected move is not the numbing scream or enemy summon move (this applies even if this is a reroll from selecting the enemy summon while this enemy isn't the last remaining `enemydata`)
- This enemy doesn't already the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- A 31% RNG check passes (41% instead if hardmode is true)

Here is the logic if all of the above are fufilled:

- animstate set to 100
- `Charge7` sound plays
- ShakeSprite called with an intensity of (0.1, 0.0, 0.0) and 45.0 frametimer
- Yield for 0.75 seconds
- animstate set to 2 (`Jump`)
- y `spin` set to 20.0
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 2 main turn with effect
- Yield for 0.5 seconds
- `spin` zeroed out
- Yield for 0.15 seconds

## Move 1 - Numbing scream
A party wide numbing scream attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `Charge7` sound plays
- Camera moves to look near this enemy
- animstate set to 100
- ShakeSprite called with intensity (0.1, 0.0. 0.0) and 75.0 frametimer
- Yield for 0.75 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (-0.5, 3.0, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 101
- `Roar3` sound plays with 0.8 pitch
- ShakeScreen called with 0.2 ammount and 0.1 time
- PartyDamage 1 call happens
- Yield for 0.5 seconds

## Move 2 - Swipe attack
A single target swipe attack that may hit twice.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|2|Happens only if at least one of the following is true:<ul><li>`commandsuccess` is true after DoDamage 1 (ignores FRAMEONE)</li><li>[HPPercent](../../Actors%20states/HPPercent.md) is less than 0.6</li></ul>|This enemy|The selected `playertargetID`|3|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with 2.0 multiplier, 1 (`Walk`) as walkstate and 103 as stopstate
- Yield all frames until `forcemove` is done
- animstate set to 109
- `Growl` sound plays
- Yield for 0.65 seconds
- `Growl2` sound plays
- animstate set to 105
- DoDamage 1 call happens
- If `commandsuccess` is true (blocked, ignores FRAMEONE) or [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.6:
    - Yield for 0.3 seconds
    - animstate set to 106
    - Yield for 0.2 seconds
    - `Growl2` sound plays with 1.1 pitch
    - DoDamage 2 call happens
- Yield for 0.5 seconds

## Move 3 - Tail slash
A single target tail slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|5|null|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.1, 0.0, -0.1) with 2.0 multiplier, 1 (`Walk`) as walkstate and 103 as stopstate
- Yield all frames until `forcemove` is done
- animstate set to 109
- `Growl3` sound plays
- A random number is generated between 0.6 and 0.8
- ShakeSprite called with intensity (0.1, 0.0, 0.0) with a frametimer of 60.0 * the random number
- Yield for an amount of seconds being the random number
- animstate set to 110
- `HugeHit` sound plays
- `Slash3` sound plays
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't block, ignores FRAMEONE), `enemybounce` is set to a new array with one element being a new BounceEnemy call with `playerdata[playertargetID]` using routineid 0 for 1 bounce with 1.2 multiplier
- Yield for 0.2 seconds
- FlipSimple called
- Yield for 0.5 seconds
- Yield all frames until all `enemybounce` are done
- FlipSimple called
- FlipAngle called with setangle
- Yield for a frame

## Move 4 - Grab attack
A single target grab attack that may hit twice.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|null|null|`commandsuccess`|
|2|Happens only if `commandsuccess` is false after DoDamage 1 (ignores FRAMEONE)|This enemy|The selected `playertargetID`|2|null|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (5.0, 0.0, -0.1) with 2.0 multiplier, 107 as walkstate and stopstate
- Yield all frames until `forcemove` is done
- `Step` sound plays on loop using `sounds[9]` with 1.3 pitch
- ShakeSprite called with intensity (0.1, 0.0, 0.0) with a frametimer of 50.0
- Over the course of 41.0 frames, this enemy moves to + 2.0 in x via a SmoothLerp
- Yield for 0.35 seconds
- Over the course of 43.5 frames (36.0 frames instead if hardmode is true), this enemy moves to (-20.0, 0.0, same z position) via a lerp and `sounds[9]` volume changes to 0.0 via a lerp calculated over half of the amount of frames. On the first frame this enemy x position becomes lower than `playertargetentity` x position:
    - DoDamage 1 call happens
    - If `commandsuccess` is false (didn't blocked, ignores FRAMEONE):
        - `Drag` sound plays
        - animstate set to 108
        - `playertargetentity` gets childed to this enemy
        - `playertargetentity` animstate set to 11 (`Hurt`)
        - `playertargetentity` local position set to (-2.35, 0.35, -0.1)
        - `playertargetentity` z angle set to -90.0
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Over the course of 43.5 frames (36.0 frames instead if hardmode is true), this enemy moves from (20.0, 0.0, same z position) to starp via a lerp and `sounds[9]` volume changes to MainManager.`soundvolume` via a lerp calculated over half of the amount of frames
- `sounds[9]` stops looping
- If `commandsuccess` was false earlier when DoDamage 1 call happened:
    - animstate set to 111
    - `playertargetentity` gets childed to the `battlemap`
    - `sprite` y angle set to -180.0
    - y `spin` set to 20.0
    - `Toss8` sound plays
    - `Toss9` sound plays
    - Over the course of 61.0 frames, `playertargetentity` moves to the position they were at before this action via a BeizierCurve3 with a ymax of 12.0. Before each frame yield, `playertargetentity` animstate set to 11 (`Hurt`)
    - `playertargetentity`'s `spin` zeroed out
    - `Thud3` sound plays
    - DeathSmoke particles plays at `playertargetentity` position with a size of 2.0x
    - ShakeScreen called with an ammount of 0.2 and 0.8 time
    - DoDamage 2 call happens
    - Yield for 0.75 seconds

## Move 5 - [Weevil](Weevil.md) summon
Summons a new [Weevil](Weevil.md) enemy. No damages are dealt.

### Logic sequence

- animstate set to 100
- Yield for 0.75 seconds
- animstate set to 102
- `Whistle2` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new [Weevil](Weevil.md) enemy with type `Offscreen` at this enemy position + (3.0, 0.0, 0.5)
- Yield for 1.25 seconds
- animstate set to 0 (`Idle`)
- Yield all frames until `checkingdead` is null (the coroutine completed)
