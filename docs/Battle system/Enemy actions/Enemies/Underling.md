# `Underling`

## Assumptions
It is assumed that this enemy gets loaded with a [isdefending](../../Actors%20states/Enemy%20features.md#isdefending) of -1 so their `hitaction` logic works.

## [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) special logic
Before this enemy is loaded, it's possible that StartBattle overrides it to a `GoldenSeedling`. See the documentation to learn more.

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### Logic sequence

- animstate set to 23 (`Chase`)
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called with the `Exclamation` emote with a time of 35
- `Wam` sound plays
- `checkingdead` set to a new [Dig](../Dig.md) call on this enemy which changes its [position](../../Actors%20states/BattlePosition.md) to `Underground` (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null

## Move selection
3 moves are possible:

1. A single target tackle attack
2. A single target headbonk attack
3. A single target undergound strike attack

Move 3 is always used (and can only be used) when [position](../../Actors%20states/BattlePosition.md) is `Underground`.

As for other moves, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|51%|
|2|49%|

## Move 1 - Tackle attack
A single target tackle attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to new coroutine called SeedlingTackle which will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` is null

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

## Move 2 - Headbonk attack
A single target headbonk attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` set to a new SeedlingHeadbonk coroutine with 2 damage, actionid as the attacker id, no property and without skipwalk. This coroutine sets `checkingdead` to null when completed
- Yield all frames until `checkingdead` is null

Here is what SeedlingHeadbonk effectively does with those parameters:

- Camera moves to look near `playerdata[playertargetID]`
- animstate set to 23 (`Chase`)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.2) with a multiplier of 2.0
- Yield all frames until `forcemove` is done
- `Jump` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.3 seconds
- Yield all frames until `onground` is true
- Camera moves to look near `playertargetID`
- Yield a frame
- animstate set to 100
- `Turn` sound plays
- Over the course of 35.0 frames (but visually occurs as if it was 32.0 frames with the last 3 being padding), this enemy moves to `playertargetentity` position + (0.25, 2.8, -0.1) via a BeizierCurve3 with a ymax of 5.5 while the z angle gets a LerpAngle done to 180.0
- DoDamage 1 call happens
- Yield for 0.1 seconds
- `sprite` angles zeroed out
- animstate set to 1 (`Walk`)
- `Turn2` sound plays
- Over the course of 25.0 frames, this enemy moves to its current position + 2.0 in x with the y being 0.0 via a BeizierCurve3 with a ymax of 3.0
- `checkingdead` is set to null which signals the caller that this coroutine completed

## Move 3 - Undergound strike attack
A single target undergound strike attack. [position](../../Actors%20states/BattlePosition.md) is set to `Ground` after.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### About incorrect [Poison](../../Damage%20pipeline/AttackProperty.md) property logic
While the DoDamage call features a `Pierce` property, the move ends up doing an incorrect version of the `Poison` property logic due to the [single property shortcoming](../../Damage%20pipeline/Known%20design%20issues.md#it-is-impossible-to-call-dodamage-using-multiple-attackproperty). Here are the differences:

- On FRAMEONE, regular blocking will incorrectly prevent the condition from being inflicted which doesn't happen normally
- The amount of turns to inflict is 3 instead of 2
- The [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition on the target won't prevent the condition to be inflicted when it normally would have
- The `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md) does nothing when equipped on the target when it should have inflicted the condition back to this enemy
- The `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) won't work despite the attack being physical
- The condition infliction can still happen if the target has the [Numb](../../Actors%20states/BattleCondition/Numb.md) while having the `ShockTrooper` [medal](../../../Enums%20and%20IDs/Medal.md) equipped
- The `Poison` sound is not played when it should have been

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to a new UndergroundStrike coroutine with 2 damage from this enemy with a [poison](../../Damage%20pipeline/AttackProperty.md) property without staying underground
- Yield all frames until `checkingdead` goes to null (the coroutine is done)

This is effectively what the coroutine does:

- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- `Digging` sound plays on loop using `sounds[9]` with 1.1 pitch and 0.75 volume
- Over the course of 100.0 frames, position moves to `playertargetentity`'s position + -0.25 in z
- Yield for 0.25 seconds
- `sounds[9]` stopped
- `DigPop` sound plays
- DoDamage 1 call happens
- If all of the following conditions are fufilled, [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called to inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition to `playerdata[playertargetID]` for 3 main turns (NOTE: this has a lot of caveats, see the section above for details):
    - `commandsuccess` is false (didn't blocked, ignores FRAMEONE)
    - `playerdata[playertargetID]`'s `poisonres` is less than 100 (it's not immune)
    - A `poisonres` resistance check passes
    - `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition
- y `spin` set to 20.0
- `digging` set to false
- scale reset to `startscale`
- animstate set to 100
- `DirtExplodeLight` particle plays at this enemy position
- ShakeScreen with 0.15 ammount and 0.2 time
- Over the course of 50.0 frames, position moves to `playertargetentity` + 2.0 in x using a BeizierCurve3 with a ymax of 6.0
- `spin` zeroed out
- [position](../../Actors%20states/BattlePosition.md) set to `Ground`
- Yield for 0.25 seconds
- `checkingdead` set to null to signal DoAction that the coroutine completed
