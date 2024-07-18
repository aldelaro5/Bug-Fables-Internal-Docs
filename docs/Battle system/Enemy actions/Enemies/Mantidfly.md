# `Mantidfly`

## Asumptions
It is assumed that this enemy gets loaded with more than 1 `moves` because the logic assumes this enemy has multiple actor turns per main turn.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the aerial dash move, the amount of frames this enemy takes to move changes to 30.0 frames from 35.5 frames
- In the sky drop move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the sky drop move, the amount of frames the drop takes changes to 34.0 frames from 38.0 frames

## Move selection
3 moves are possible:

1. A single target slash attack
2. A single target aerial dash attack
3. A single target sky drop that the player may prevent from dealing damages

The decision on which move to use is based on odds, but the odds changes depending on if `cantmove` is 0 or not (0 means this enemy is taking their last available actor turn of the current main turn). Here are the odds in either situations:

|Move|Odds when `cantmove` isn't 0|Odds when `cantmove` is 0|
|---:|-------------|--------|
|1|1/2|2/7|
|2|1/2|2/7|
|3|Never used|3/7|

## Pre move logic
The following logic always happen before using a move:

- `initialheight` set to 2.0
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- If [position](../../Actors%20states/BattlePosition.md) is `Ground`:
    - `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to change the [position](../../Actors%20states/BattlePosition.md) to `Flying`
    - Yield all frames until `checkingdead` is null (the coroutine completed)

## Move 1 - Slash attack
A single target slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- Camera moves to look near `playerdata[playertargetID]`
- animstate set to 1 (`Walk`)
- Over the course of 41.0 frames:
    - `height` changes to 0.0 via a lerp calculated over 60.0 frames
    - This enemy moves to `playertargetentity` position + (1.5, 0.0, -0.1) via a SmoothLerp
    - Before each frame yield when `height` is above 0.0 and this enemy's `sound` isn't playing, `BugWing` sound plays on this enemy's `sound`
- animstate set to 100
- `Growl2` sound plays with 1.1 pitch
- Yield for 0.5 seconds
- `Slash3` sound plays
- DoDamage 1 call happens
- Yield for 0.5 seconds
- [position](../../Actors%20states/BattlePosition.md) is set to `Ground`

## Move 2 - Dash attack
A single target aerial dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|[Sleep](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- animstate set to 101
- `BugWingFast` sound plays
- Over the course of 31.0 frames, this enemy moves to + 2.0 in x via a SmoothLerp. Before each frame yield, `height` decreases by BeizierFloat of 1.0 (will smoothly decrease up to 0.5 then back to no decrease)
- ShakeSprite called with an intensity of 0.1 and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 104
- Camera moves to look near `playerdata[playertargetID]`
- `Toss4` sound plays
- Over the course of 35.5 frames (30.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (-3.0, 0.0, -0.1) via a BeizierCurve3 with a mid of `playertargetentity` position + (0.0, 0.0 - this enemy `height`, -0.1). On the first frame this enemy x position becomes lower than the `playertargetentity` x position, DoDamage 1 call happens
- Yield for 0.25 seconds

## Move 3 - Sky drop
A single target sky drop that the player may prevent from dealing damages.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Only happens if `barfill` is less than 0.975 (almost filled) after DoCommand 1|This enemy|The selected `playertargetID`|5|null|null|`commandsuccess`|

### Logic sequence

- If `height` is above 0.1, `BugWing` sound plays via PlayMoveSound on loop using `sounds[9]`
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with a 2.5 multiplier
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- animstate set to 101
- ShakeSprite called with intensity of 0.1 and 30.0 frametimer
- Yield for 0.5 seconds
- `Grow1` sound plays
- If `height` is above 0.1, `BugWing` sound plays via PlayMoveSound on loop using `sounds[9]`
- Over the course of 11.0 frames, this enemy moves to `playertargetentity` position + (0.5, 0.0, -0.1) via a lerp and their `height` changes to 0.0 via a lerp
- animstate set to 102
- `playertargetentity` gets childed to this enemy `sprite`
- Yield for 0.25 seconds
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the `TappingKey` command's help)
- DoCommand 1 call happens
- Yield all frames until `doingaction` goes to false, but only waiting up to 166.0 frames. Over the course of this wait, this enemy moves to + 6.5 in y via a lerp
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- Yield for 0.25 seconds
- `sounds[9]` stopped
- If `barfill` is 0.975 or higher (the bar was almost filled), the attack is cancelled with the following:
    - `playertargetentity` animstate set to 13 (`BattleIdle`)
    - `playertargetentity` gets childed to `battlemap`
    - This enemy `height` set to its value before this action
    - This enemy y position gets decreased by the `height` value before this action
    - This enemy animstate set to 0 (`Idle`)
    - This enemy `flyinganim` set to true
    - Yield all frames until `playertargetentity` is `onground`
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - This action is over early
- Otherwise (the bar wasn't almost filled):
    - Over the course of 51.0 frames, this enemy moves to 15.0 in y via a lerp
    - animstate set to 103
    - y `spin` set to 30.0
    - `sprite` z angle set to 180.0
    - `Fall2` sound plays
    - Over the course of 38.0 frames (34.0 frames instead if hardmode is true), this enemy moves to 1.25 in y via a lerp
    - ShakeScreen called with an ammount of (0.2, 0.0, 0.0) with a time of 1.0 with dontreset
    - `Thud3` sound plays with 1.1 pitch
    - `impactsmoke` partciles plays at this enemy position
    - DoDamage 1 call happens
    - `spin` zeroed out
    - `sprite` z angle set to 180.0
    - Yield for 0.25 seconds
    - `sprite` angles zeroed out
    - Yield for a frame
    - `playertargetentity` gets childed to the `battlemap`
    - animstate set to 0 (`Idle`)
    - Over the course of 31.0 frames, this enemy moves to + 2.0 in x via a lerp and their `height` changes to the value it had before this action
