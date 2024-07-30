# `BeeBot`

## Assumptions
It is assumed that this enemy gets loaded with the `BeeBot` [animid](../../../Enums%20and%20IDs/AnimIDs.md) which is needed for the [CheckSpecialID](../../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) to work as it is needed for correct move selection logic.

It is also that this enemy gets laoded with a [onhitaction](../../Actors%20states/Enemy%20features.md#onhitaction) of 1 so their `hitaction` logic works.

## randomposafter set to true
This enemy action always sets the randomposafter local to true except in a `hitaction` logic. This means that in [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#no-fled-enemy-post-action) when no enemy fled, this enemy's [position](../../Actors%20states/BattlePosition.md) will be randomly determined between `Ground` and `Flying`.

## `data` usage
On [CheckSpecialID](../../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) for this enemy, if `data` is null or empty, it's initialised to be 1 element with a starting value being 0 or 1 determined randomly with uniform probabilities on top of calling [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md).

- `data[0]`: This value determines the move that will be used. A value of 0 means the dash attack move will be used and a value of 1 means the honey throw move will be used. It is meant to be toggled between 0 and 1 during the `hitaction` logic. Since the starting value is random between 0 and 1 inclusive, it means the starting move slated for selection is random

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the honey throw move, 2 projectiles will be thrown instead of 3
- In the honey throw move, the speed of each projectile changes to 0.03 (~33.333333334 frames of movement) from 0.025 (40.0 frames of movement)

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves (but the post move logic will still occur)

### Logic sequence

- `Jump` sound plays with 1.1 volume
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.5 seconds
- `BeeBotModeSwitch` sound plays
- animstate set to 100 + `data[0]` (normally, 100 or 101)
- Yield for a frame
- `data[0]` toggles: if it was 0, it's set to 1 and if it was 1, it's set to 0
- Yield for 0.6 seconds
- animstate set to 0 (`Idle`)
- [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md) called

## Move selection
2 moves are possible:

1. A single target dash attack
2. A single target multiple honey projectiles throw

The decision of which moves to use entirely depends on the value of `data[0]`: Move 1 is used if it's 0 and move 2 is used if it's 1. The starting value is random between 0 and 1, but it toggles between the 2 during the `hitaction` logic.

## Pre move logic
The following logic always happen before a move (this excludes `hitaction` where this logic won't happen):

- If [position](../../Actors%20states/BattlePosition.md) is `Ground`, it is set to flying with the following logic:
    - Over the course of 21.0 frames, `height` changes from `minheight` to 2.0 via a lerp
    - [position](../../Actors%20states/BattlePosition.md) is set to `Flying`

## Post move logic
The following logic always happen at the end of the action (even if a `hitaction` was done):

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 multiplier using 0 (`Idle`) as both walkstate and stopstate
- Yield all frames until `forcemove` is done

## Move 1 - Dash attack
A single target dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moved to look near the midpoint between `playertargetentity` and this enemy
- `height` set to 0.0
- This enemy moves 2.0 in y
- `sprite` z angle set to 20.0
- `Spin7` sound plays
- Over the course of 31.0 frames, this enemy moves to its postion + (1.5, 3.0, -0.25) via a BeizierCurve3 with a ymax of -0.75
- `Charge10` sound plays
- ShakeSprite called with intensity (0.2, 0.0, 0.0) with 30.0 frametimer
- Yield for 0.5 seconds
- `Prefabs/Particles/WalkDust` instantiated childed to the `sprite` with a local position of Vector3.zero and the following adjustements:
    - main.startLifetime: 1.5 constant curve
    - main.startSize: 1.2
    - emission.rateOverTime: 15.0 constant curve
- Camera moves to look near `playertargetentity`
- `Shot2` sound plays
- Over the course of 31.0 frames, this enemy moves to `playertargetentity` position + (-2.5, 2.0, -0.1) via a BeizierCurve3 with a ymax of -1.0. After the first frame that this enemy x position becomes equal or lower than the `playertargetentity` x position, DoDamage 1 call happens
- `Prefabs/Particles/WalkDust` moved offscreen to 9999.0 in y then destroyed in 3.0 seconds
- Yield for 0.25 seconds
- `sprite` angles reset to the value before this action
- `height` reset to the value before this action

## Move 2 - Honey projectiles throw
A single target multiple honey projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 3 times (2 times instead if hardmode is true)|1|[Numb](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target is the same for each calls)|A new sprite object rooted using the `projectilepsrites[10]` sprite (honey projectile) positioned at this enemy's `model`'s first child's third child (its minigun) + (-0.6, -0.25, -0.1) with a scale of 0.3x, a ShadowLite that is SetUp with 0.4 opacity and 0.5 size and a DialogueAnim with `targetscale` of 0.75x and a 0.1 `shinkspeed`|0.025 (40.0 frames of movement) if hardmode is false, 0.03 if it's true (~33.333334 frames of movement)|0.0|null|`ElecFast`|null|null|Vector3.zero|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near (-2.0, 0.0, 0.0)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (1.0, 0.0, -0.5) with 2.0 multiplier using 0 (`Idle`) as both walkstate and stopstate
- Yield all frames until `forcemove` is done
- The minigun's SpriteRenderer is obtained which is its `model`'s first child's third child
- The minigun is enabled
- Over the course of 16.0 frames, the y local position of the minigun changes to -0.5 (the x is always set to 0.0 and the z to 0.05)
- Yield for 0.25 seconds
- The SpriteBounce of the minigun is enabled
- The amount of projectiles to throw is determined. It is always 3 (2 instead if hardmode is true) and a new SpriteRenderer array is initialised to hold each projectile
- For each projectile to throw:
    - `Spit` sound plays
    - A new sprite object is created rooted using the `projectilepsrites[10]` sprite (honey projectile) positioned at this enemy's `model`'s first child's third child (its minigun) + (-0.6, -0.25, -0.1) with a scale of 0.3x, a ShadowLite that is SetUp with 0.4 opacity and 0.5 size and a DialogueAnim with `targetscale` of 0.75x and a 0.1 `shinkspeed`
    - Yield for a frame
    - HitPart partiles plays at the projectile position
    - `Shot` sound plays
    - Projectile 1 call happens
    - Over the course of 11.0 frames, this enemy moves + 0.5 in x via a BeizierCurve3 with a ymax of -0.25
    - Yield for 0.1 seconds
    - Over the course of 11.0 frames, this enemy moves - 0.5 in x via a BeizierCurve3 with a ymax of -0.1
    - position set to the one it had before throwing this projectile
The minigun's SpriteBounce is disabled
- Yield all frames until all projectiles array elements are null (meaning all Projectile 1 calls completed)
- Yield for 0.5 seconds
- Over the course of 16.0 frames, the y local position of the minigun changes to 0.0 (the x is always set to 0.0 and the z to 0.05)
- The minigun is disabled
