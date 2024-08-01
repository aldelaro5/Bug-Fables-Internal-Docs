# `Plumpling`

## [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) special logic
Before this enemy is loaded, it's possible that StartBattle overrides it to a `GoldenSeedling`. See the documentation to learn more.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the jump attack move, the amount of frames this enemy takes to reach its target changes to 38.25 frames from 51.0 frames

## Move selection
1 moves is possible:

1. A single target jump attack

Move 1 is always used.

## Move 1 - Jump attack
A single target jump attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- `checkingdead` is set to a new HeavyJump coroutine for this enemy with 4 damage and 60.0 speed (45.0 if hardmode is true)
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine effectively ends up doing:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- `ThumpSoft` sound plays on loop using `sounds[9]` with 0.65 pitch and 0.65 volume using PlayMoveSound
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.1)
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- Yield for 0.3 seconds
- `Jump` sound plays with 0.8 pitch
- `Turn` sound plays with 0.7 pitch
- Over the course of 51.0 frames (38.25 frames instead if hardmode is true), this enemy moves to `playertargetentity` position -0.1 in z via a BeizierCurve3 with a ymax of 5.5 scaled over the course of 60.0 frames (45.0 frames instead if hardmode is true). Before each frame yield, animstate is set to 2 (`Jump`) until halfway 30.0 frames passed (27.5 frames instead if harmode is true) where it's set to 3 (`Fall`) instead
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE), ShakeScreen is called with ammount (0.075, 0.3, 0.0) and 0.75 time with dontreset
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE):
    - Over the course of 9.0 frames (16.75 frames instead if hardmode is true), this enemy moves to `playertargetentity` position -0.1 in z via a BeizierCurve3 with a ymax of 5.5. Before each frame yield, animstate is set to 3 (`Fall`)
    - `impactsmoke` particles plays at `playertargetentity` position
    - `Thud4` sound plays
    - `HugeHit3` soud plays
- Otherwise (blocked, ignores FRAMEONE):
    - Over the course of 41.0 frames, this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 3.0. Before each frame yield, animstate is set to 3 (`Fall`)
    - `Thud4` sound plays
    - `impactsmoke` particles plays at `playertargetentity` position
    - ShakeScreen is called with ammount (0.05, 0.2, 0.0) and 0.5 time with dontreset
    - animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
- `checkingdead` is set to null informing the caller that the coroutine completed
