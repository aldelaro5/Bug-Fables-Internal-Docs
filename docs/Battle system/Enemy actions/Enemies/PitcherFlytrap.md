# `PitcherFlytrap`

## Assumptions
It is assumed that this enemy is loaded with the `PicherSummon` [animid](../../../Enums%20and%20IDs/AnimIDs.md) as this action assumes the presence of `extra[0]` which is initialised by [CheckSpecialID](../../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) only with this animid.

## HPBarOnOther special logic
This enemy has special logic in a method called HPBarOnOther which is used by [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) to know if the `hpbar` should be shown despite this enemy not being spied yet. It returns true if [Pitcher](Pitcher.md) was spied which is needed because it's not possible under normal gameplay to spy this enemy, but it is possible to spy `Pitcher`. This means that this logic allows this enemy's `hpbar` to be displayed by spying `Pitcher`.

## Move selection
1 moves is possible:

1. A single target bite attack

Move 1 is always used.

## Move 1 - Bite attack
A single target bite attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- `Chew` sound plays with 0.75 pitch
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 101
- DoDamage 1 call happens
- Camera moves to look near `playertargetentity`
- `Bite2` sound plays with 0.9 pitch
- For each frames on the next 30.0 frames, `extra[0]`'s parent's parent's parent (their arm's middle bone) position increases by (3.75, 1.5, -0.1)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Over the course of 3.0 frames, `extra[0]`'s parent's parent's parent (their arm's middle bone) moves to its position before this action via a lerp
-  `extra[0]`'s parent's parent's parent (their arm's middle bone) local position is set to its value before this action
