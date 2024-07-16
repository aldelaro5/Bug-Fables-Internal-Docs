# `JumpingSpider`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The move selection odds changes to favor the high jump attack where it becomes 40% chance to be used from 25%
- In the small jump attack move, the amount of frames this enemy takes to reach its target changes to 41.0 frames from 48.0
- In the high jump attack move, the amount of frames this enemy takes to reach its target changes to 66.0 frames from 81.0

## Move selection
2 moves are possible:

1. A single target small jump attack that may steal a standard [item](../../../Enums%20and%20IDs/Items.md)
2. A single target high jump attack

The decision of which move to use depends on these odds which changes depending on hardmode:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|----|----|
|1|75%|25%|
|2|60%|40%|

## Move 1 - Small jump attack
A single target small jump attack that may steal a standard [item](../../../Enums%20and%20IDs/Items.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)
- Camove moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.1) with a walkstate of 23 (`Chase`) and a stopstate of 0 (`Idle`)
- Yield all frames until `forcemove` is done and `onground`
- `Turn` sound plays
- Over the course of 48.0 frames (41.0 frames instead if hardmode is true), thie enemy moves to `playertargetentity` position + (-0.25, 1.5, -0.1) via a BeizierCurve3 with a ymax of 5.0. Before each frame yield, the animstate is set to 2 (`Jump`) except for the second half of the movement where it's set to 3 (`Fall`)
- DoDamage 1 call happens
- If `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, [StealItem](../StealItem.md) is called from this enemy without always and with noanim
- [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md#updateanimspecific) called on this enemy
- `Turn2` sound plays
- Over the course of 41.0 frames, this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 3.0
- animstate set to 0 (`Idle`)
- Yield for 0.25 seconds

## Move 2 - High jump attack
A single target high jump attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|5|null|[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)
- Camove moves to look near the midpoint between this enemy and `playerdata[playertargetID]`
- ShakeSprite called with an intensity of 0.15 and frametimer of 45.0
- animstate set to 1 (`Walk`)
- `BMCharge` sound plays with 1.2 pitch
- Over the course of 46.0 frames, the `rotater` scale is set to increase by 15% in x and decreased by 35% in y via a lerp
- animstate set to 2 (`Jump`)
- y `spin` set to 30.0
- `rotater` scale set to its value before this action
- `Turn` sound plays with 0.75 pitch
- Over the course of 81.0 frames (66.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (0.25, 2.0, -0.1) via a BeizierCurve3 with a ymax of 15.0. Before each frame yield on the second half of the movement, animstate is set to 3 (`Fall`) and after 75% of the movement, the camera moves to look near `playerdata[playertargetID]`
- `spin` zeroed out
- ShakeScreen called with an ammount of 0.2 and time of 0.75 with dontreset
- DoDamage 1 called
- `BigHit` sound plays
- `impactsmoke` particles plays at `playertargetentity` position + 0.65 in y
- Over the course of 41.0 frames, this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 3.0
- animstate set to 0 (`Idle`)
- Yield for 0.25 seconds
