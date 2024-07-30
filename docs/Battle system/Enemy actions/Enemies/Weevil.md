# `Weevil`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The enemy party member chewing move, if all other conditions applies, has a 8/10 chance to occur instead of 5/10
- In the enemy party member chewing move, this enemy now gets healed their `hp` to their `maxhp` instead of 5 `hp`
- In the player party member chewing move, the odds to gain an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) conditions if it wasn't already inflicted changes to 5/10 from 3/10

## Move selection
2 moves are possible:

1. A single target chewing attack
2. Chews on the first [Seedling](Seedling.md), [AngryPlant](AngryPlant.md) or [FlyTrap](FlyTrap.md) among the enemy party which defeats them, but drains their `hp`

Move 2 is always used if all of the following conditions are fufilled:

- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.65
- A 5/10 RNG check passes (8/10 instead if hardmode is true)
- A [Seedling](Seedling.md), [AngryPlant](AngryPlant.md) or [FlyTrap](FlyTrap.md) is present among the enemy party
- The applicable enemy party member's [position](../../Actors%20states/BattlePosition.md) is `Ground`

If these conditions aren't all fufilled, move 1 is always used instead.

## Pre move logic
Before the move, the first `enemydata` index who is a [Seedling](Seedling.md), [AngryPlant](AngryPlant.md) or [FlyTrap](FlyTrap.md) is obtained if one exists. This is done to both check if move 2 can be used and also to get the right `enemydata` index for it.

## Move 1 - Chewing attack on player party member
A single target chewing attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- If this enemy doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition and a 3/10 RNG check passes (5/10 instead if hardmode is true):
    - Camera moves to look near this enemy zoomed in
    - `WeevilAwoo` sound plays
    - animstate set to 100
    - Yield for 0.8 seconds
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 0 (red up arrow)
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 2 main turns
    - Yield for 0.75 seconds
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentitiy`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.0, 0.0, -0.1) at 2.0 multiplier using 23 (`Chase`) as walkstate and 103 as stopstate
- Yield all frames until `forcemove` is done
- `playertargetentity`'s y `spin` set to 15.0
- Done 4 times:
    - position set to (`playertargetentity` x position +/- 1.5, 0.0, `playertargetentity` z position - 0.1). The x position offset is -1.5 on the first and third time while it's +1.5 on the second and fourth time
    - [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) `playertargetentity`
    - `sprite` angles set to its FlipAngle
    - `Chew` sound plays with volume 1.1 + 0.1 * the 0 based index of this loop
    - For the next 10.0 frames, `playertargetentity` animstate is set to 11 (`Hurt`) unless `blockcooldown` hasn't expired where it's set to 24 (`Block`) instead
- `playertargetentity`'s `spin` zeroed out
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `flip` set to true
- DoDamage 1 call happens
- Yield for 0.2 seconds
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp at 2.0 multiplier using 23 (`Chase`) as walkstate and 0 (`Idle`) as stopstate
- Yield all frames until `forcemove` is done
- `flip` set to false

## Move 2 - Chewing attack on [Seedling](Seedling.md), [AngryPlant](AngryPlant.md) or [FlyTrap](FlyTrap.md) enemy party member
Chews on the first [Seedling](Seedling.md), [AngryPlant](AngryPlant.md) or [FlyTrap](FlyTrap.md) among the enemy party which defeats them, but drains their `hp`. No damages are dealt to the player party and for the enemy party members, nothing happens through the damage pipeline (everything is done manually).

### Logic sequence

- `playertargetentity` is set to the targetted enemy party member
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called with the `Exclamation` emote with a time of 30
- Yield for 0.6 seconds
- Camera moves to look near `playertargetentitiy`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.0, 0.0, -0.1) at 2.0 multiplier using 23 (`Chase`) as walkstate and 103 as stopstate
- Yield all frames until `forcemove` is done
- `playertargetentity`'s y `spin` set to 15.0
- Done 4 times:
    - position set to (`playertargetentity` x position +/- 1.5, 0.0, `playertargetentity` z position - 0.1). The x position offset is -1.5 on the first and third time while it's +1.5 on the second and fourth time
    - [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) `playertargetentity`
    - `sprite` angles set to its FlipAngle
    - `Chew` sound plays with volume 1.1 + 0.1 * the 0 based index of this loop
    - For the next 10.0 frames, `playertargetentity` animstate is set to 11 (`Hurt`)
- `playertargetentity`'s `spin` zeroed out
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `flip` set to true
- `Damage0` sound plays
- HitPart particles played at `playertargetentity` + Vector3.up
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy for 5 `hp` (to its `maxhp` instead if hardmode is true which heals them to full)
- The targeted enemy party member's `hp` is set to 0
- The targeted enemy party member's `exp` is set to 0
- The targeted enemy party member's `money` is set to 0
- Yield for 0.2 seconds
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp at 2.0 multiplier using 23 (`Chase`) as walkstate and 0 (`Idle`) as stopstate
- Yield all frames until `forcemove` is done
- `flip` set to false
- StartDeath is called on the targetted enemy party member which ends up setting its `deathcoroutine` to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call with activatekill followed by a frame yield.

NOTE: The StartDeath call doesn't do everything that CheckDead was supposed to do outside of the bestiary change and rewards setup (which were undesired already). Here are what is omitted to be done:

- DestroyConditionIcons is not called on the targetted enemy party member's battleentity. This allows them to be rendered for a very brief moment before their death and eventual destruction which normally wouldn't happen since they would be destroyed before their death process happens
- If the enemy party member had any `holditem`, they aren't dropped via [DropItem](../../Actors%20states/Enemy%20party%20members/DropItem.md)
- If the enemy party member had an [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath), it is not triggered

However, if the enemy party member's [deathtype](../../Actors%20states/Enemy%20features.md#deathtype) doesn't end with their destructions, some information that was supposed to be cleared will be persisted:

- `charge` isn't zeroed out
- [condition](../../Actors%20states/Conditions.md) aren't cleared
