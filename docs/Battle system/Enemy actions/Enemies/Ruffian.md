# `Ruffian`

## Assumptions
It is assumed that enemy is loaded with more than 1 `moves` as there is a distinction in the move selection if they are performing their last actor turn of the main turn or not.

It is also assumed that this eneny has the `Ruffian` [animid](../../../Enums%20and%20IDs/AnimIDs.md) as it is required for `extra[3]` to be present which is normally initialised from [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md#updateanimspecific) in the logic specific to the `Ruffian` animid.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the kick attack move, the damage of the BasicAttack call changes to 3 from 4 (under normal gameplay, this enemy has a `hardatk` of 1 which would make it do the same on this move regardless of hardmode being true or not)
- In the ball and chain attack move, the property of the DoDamage call is [DefDownOnBlock](../../Damage%20pipeline/AttackProperty.md) instead of null

## Move selection
4 moves are possible:

1. A single target kick attack
2. A single target arms slam attack
3. Prepares themselves for a big attack next actor turn by setting their `charge` to 1
4. A single target ball and chain attack

Move 4 is always (and only) used when `charge` is higher than 0 meaning move 3 was used previously.

As for the other moves, the decision of which move to use is based on odds, but the odds changes depending if `cantmove` is 0 or not (0 means that this enemy is performing their last actor turn of the main turn). Here are the odds in either situations:

|Move|Odds when `cantmove` < 0|Odds when `cantmove` is 0|
|---:|-----|-----|
|1|1/4|1/2|
|2|1/4|1/2|
|3|2/4|Never used|

## Move 1 - Kick attack
A single target kick attack.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|102|103|4 (3 instead if hardmode is true)|(1.5, 0.0, -0.1)|null|0.0|0.5 seconds|`RuffianBindingShake,RuffianBindingSwing`|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (the BasicAttack 1 call completed)

## Move 2 - Arms slam attack
A single target arms slam attack.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|100|101|3|(1.5, 0.0, -0.1)|[Flip](../../Damage%20pipeline/AttackProperty.md)|0.075|0.5 seconds|`RuffianKickShake,RuffianKickSwing`|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (the BasicAttack 1 call completed)

## Move 3 - Prepares a big attack with 1 `charge`
Prepares themselves for a big attack next actor turn by setting their `charge` to 1. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called (it won't do anything for this move)
- `Charge7` sound plays
- animstate set to 11 (`Hurt`)
- ShakeSprite called with 0.125 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- `StatUp` sound plays
- animstate set to 0 (`Idle`)
- `charge` set to 1
- Yield for 0.25 seconds

## Move 4 - Ball and chain attack
A single target ball and chain attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null ([DefDownOnBlock](../../Damage%20pipeline/AttackProperty.md) instead if hardmode is true)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.25, 0.0, -0.1) with 2.0 multiplier using 1 (`Walk`) as walkstate and 104 as stopstate
- Yield all frames until `forcemove` is done
- `RuffianBallShake` sound plays
- ShakeSprite called with 0.075 intensity and 40.0 frametimer
- FlipSimple called on this enemy
- Yield for 0.5 seconds
- Yield for 0.1 seconds
- animstate set to 105
- FlipSimple called on this enemy
- `RuffianBallKick` sound plays
- Over the course of 11.0 frames, `extras[3]` (this enemy's iron ball) moves to `playertargetentity` position + (0.0, 2.0, -0.1) via a BeizierCurve3 with a ymax of -2.0. On the second half of the movement, `flip` is set to true before each frame yield
- `flip` set to false
- DoDamage 1 call happens
- Yield for 0.5 seconds
