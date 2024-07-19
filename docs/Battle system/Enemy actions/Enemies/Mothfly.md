# `Mothfly`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to use the self sacrifice move when all other conditions are fufilled changes to 50% from 35%

## Move selection
3 move are possible:

1. Self sacrifices themsevles to heal a [FalseMonarch](FalseMonarch.md) or [MothflyCluster](MothflyCluster.md) by all of this enemy's `hp`
2. A single target aerial strike attack
3. A single target tackle attack, UNUSED

Move 1 is always (and only) used if all the following conditions are fufilled:

- [flags](../../../Flags%20arrays/flags.md) 400 is false (temporary global flag, but the check here is specifically meant for having this move disabled during the fight in [event](../../../Enums%20and%20IDs/Events.md) 222 which is the event related to the `Rebecca` [BoardQuest](../../../Enums%20and%20IDs/BoardQuests.md) as the event puts it to true before the fight and back to false after)
- There's at least 1 [FalseMonarch](FalseMonarch.md) or [MothflyCluster](MothflyCluster.md) in the enemy party and the first occurence of either has an [HPPercent](../../Actors%20states/HPPercent.md) below 1.0 (not at full `hp`)
- This enemy's `turnsalive` is above 0 (they're not only alive, but they also performed a complete actor turn before)
- A 35% RNG check passes (50% instead if hardmode is true)

If any of the above aren't fufilled, Move 2 is used instead.

Move 3 is never used under normal gameplay, but remains functional. This is because this move requires the [position](../../Actors%20states/BattlePosition.md) to not be `Flying`, but the pre move logic will always set it to `Flying`.

## Pre move logic
The following logic is always done at the start of the actor turn:

- `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to set the [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` set to null when completed)
- Yield all frames until `checkingdead` is null

## Move 1 - Self sacrifice
Self sacrifices themsevles to heal a [FalseMonarch](FalseMonarch.md) or [MothflyCluster](MothflyCluster.md) by all of this enemy's `hp`. No damages are dealt.

### Logic sequence

- The first occurence of a [FalseMonarch](FalseMonarch.md) or [MothflyCluster](MothflyCluster.md) in the `enemydata` is selected as the receiver of the heal
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote with 30 time
- Yield all frames until `emoticoncooldown` expires
- Camera moves to look near the receiver position
- This enemy [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) the receiver
- `Toss12` sound plays
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on this enemy to move to the receiver position + (0.0, -0.5 or 0.75 instead if the receiver has a `height` above 0.0, -0.1) with 20.0 frametime without changeanim
- Yield all frames until `forcemoving` is done
- startp is set to this enemy's `startscale`
- Over the course of 21.0 frames, this enemy's `startscale` changes to 0.0x via a lerp
- startp set to Vector3.zero
- [Heal](../../Actors%20states/Heal.md) called to heal the receiver by an amount of `hp` equal to the amount of this enemy
- `Heal` sound plays
- `HealPing` sound plays
- Yield for 0.5 seconds
- `selfsacrifice` set to true which prevents some logic to happen in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) since this enemy is set to die soon
- [CleanKill](../../Actors%20states/CleanKill.md) called on this enemy which turns this enemy into a blank actor shell placing their battleentity at startp

## Move 2 - Aerial strike attack
A single target aerial strike attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|1|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- `Toss2` sound plays via PlayMoveSound on loop using `sounds[9]` with pitch 1.1 and volume 0.5
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (2.0, 0.0, -0.15) with 1.5 multiplier and stopstate 100
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `height` set to 0.0
- y position set to the `height` value before this action
- animstate set to 11 (`Hurt`)
- `FastWoosh` sound plays
- Over the course of 1 / 0.055 frames (~18.181818 frames), position moves to its current position + 0.75 in x via a BeizierCurve3 with a ymax of -0.01
- animstate set to 100
- Yield for 0.2 seconds
- animstate set to 100
- `Turn2` sound plays
- Over the course of 1 / 0.07 frames (~14.285714 frames), position lerps to `playerdata[playertargetID]` position + (0.0, 1.0, -0.1)
- DoDamage call 1 happens
- Yield for 0.15 seconds
- `height` set to 0.2
- animstate set to 1 (`Walk`)
- Over the course of 1 / 0.045 frames (~22.22222 frames), position lerps to its position before the BeizierCurve3 movement earlier
- y position is set to 0.0
- `height` is restored to its value before this action
- Yield for 0.2 seconds
- `bobrange` reset to `startbf`
- `bobspeed` reset to `startbs`
- FlipAngle called with setangle

## Move 3 - Tackle attack, UNUSED
A single target tackle attack.

> NOTE: This move is practically UNUSED, see the move selection section above for more details.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to new coroutine called SeedlingTackle which will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` is null
- FlipAngle called with setangle

This is what the coroutine effectively does:

- animstate set to 23 (`Chase`)
- Done 2 times:
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - `Jump` sound plays with 0.75 pitch and 1.6 volume
    - Yield for 0.1 seconds
    - Yield all frames until `onground` goes to true
    - Yield a frame
- [CameraFocusTarget](../../Visual%20rendering/CameraFocusTarget.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + 3.0 in x with a 2.0 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- Yield for 0.2 seconds
- `Spin10` sound plays
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + (0.35, 0.0, -0.1) with a 2.5 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- DoDamage 1 call happens
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- `flip` set to false (with `overrideflip`)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + 2.0 in x with a 0.75 multiplier and both the current animstate as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- `flip` is set to false
- `checkingdead` set to null indicating the caller that the coroutine completed
