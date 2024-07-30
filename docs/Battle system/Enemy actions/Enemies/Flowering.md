# `Flowering`

## [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) special logic
Before this enemy is loaded, it's possible that StartBattle overrides it to a `GoldenSeedling`. See the documentation to learn more.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the enemy support move, when healing the enemy, the minimum amount of `hp` they will be healed by is 3 instead of 2

## Move selection
3 move are possible:

1. Supports another enemy party member by either healing their `hp` or inflicting them an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
2. A single target aerial strike attack
3. A single target tackle attack, UNUSED

Move 1 is always used if all the following conditions are fufilled:

- [position](../../Actors%20states/BattlePosition.md) is `Flying` (this should always be the case due to the pre move logic always setting it to `Flying`)
- This enemy isn't the only enemy party member left
- If there's exactly 2 enemy party members, the other enemy party member isn't another `Flowering`

If any of the above aren't fufilled, Move 2 is used instead.

Move 3 is never used under normal gameplay, but remains functional. This is because this move requires the [position](../../Actors%20states/BattlePosition.md) to not be `Flying`, but the pre move logic will always set it to `Flying`.

## Pre move logic
The following logic is always done at the start of the actor turn if [position](../../Actors%20states/BattlePosition.md) isn't `Flying`:

- `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to set the [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` set to null when completed)
- Yield all frames until `checkingdead` is null

## Move 1 - Enemy support
Supports another enemy party member by either healing them or inflicting them an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. No damages are dealt.

### Logic sequence

- An enemy party member other than this enemy is searched to see if any has their [HPPercent](../../Actors%20states/HPPercent.md) less than 0.6. If any exists, the first one will be selected for healing. Otherwise, a condition infliction will happen on a random enemy party member other than this enemy
- `Toss2` sound plays on loop using `sounds[9]` with 1.1 pitch and 0.5 volume using PlayMoveSound
- Camera moves to look near the targeted enemy party member
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move to the targetted enemy party member's position + their `cursoroffset` + (0.0, 0.25, -0.1) with 60.0 frametime using 1 (`Walk`) as movestate and 0 (`Idle`) as stopstate
- Yield all frames until `forcemoving` is done
- `sounds[9]` stopped
- y `spin` set to 10.0
- `Charge14` sound plays
- `Prefabs/Particles/FloweringHeal` particles instantiated rooted positioned at this enemy + their `height` in y
- Yield for 0.85 seconds
- If healing the enemy party member:
    - [Heal](../../Actors%20states/Heal.md) is called to heal the enemy party member with an `hp` amount of their `maxhp` * 0.075 clamped from 2 to their `maxhp` (clamped from 3 to their `maxhp` instead if hardmode is true)
- Otherwise (inflicting a condition on the enemy party member):
    - `StatUp` sound plays
    - If a 6/10 RNG check passes:
        - [StatEffect](../../Visual%20rendering/StatEffect.md) is called on the target enemy party member with type 0 (red up arrow)
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the target enemy party member for 2 main turns
    - Otherwise (4/10 RNG):
        - [StatEffect](../../Visual%20rendering/StatEffect.md) is called on the target enemy party member with type 1 (blue up arrow)
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the target enemy party member for 2 main turns
- Yield for 0.75 seconds
- `Prefabs/Particles/FloweringHeal` particles moved offscreen at -9999.0 in y then gets destroyed in 2.0 seconds
- `spin` zeroed out

## Move 2 - Aerial strike attack
A single target aerial strike attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|1|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- `Toss2` sound plays via PlayMoveSound on loop using `sounds[9]` with pitch 1.1 and volume 0.5
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (2.0, 0.0, -0.15) with 1.5 multiplier and stopstate 100
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `height` set to 0.0
- y position set to the `height` value before this action
- animstate set to 100
- `FastWoosh` sound plays
- Over the course of 1 / 0.055 frames (~18.181818 frames), position moves to its current position + 0.75 in x via a BeizierCurve3 with a ymax of -0.01
- animstate set to 101
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
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

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
