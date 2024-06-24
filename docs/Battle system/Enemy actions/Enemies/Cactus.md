# `Cactus`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 2 changes:

- The spike throw move has the projectile move for 25.0 frames instead of 30.0 frames
- The spin attack move has the movement before the first hit lasts for 30.0 frames instead of 50.0 frames

## Move selection
2 moves are possible:

1. A single target spike throw
2. A single target spin attack

The odds of which one is used is 50% each.

## Move 1 - spike throw
A single target spike throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy, but zoomed in
- `Charge7` sound plays
- animstate set to 102
- Yield for 0.5 seconds
- ShakeSprite called for (0.1, 0.1, 0.1) intensity and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 104
- TempSpin called for (0.0, 20.0, 0.0) for 0.5 time
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- 4 SpriteRenderers are created to be new sprite object using the instance.`projectilepsrites[8]` sprite (a small spike) near the startp with z angles being 0.0, -45.0, -45.0 and -90.0 (meaning only 3 are visible, but one is vertically aligned with the player party) Each have their normalized left vector stored in a local array which represents their heading direction (which is in a straight line away from the enemy)
- Camera moves to look near the `playertargetentity`
- `PingShot` sound plays
- Over the course of 30.0 frames (25.0 if hardmode is true), all 4 spike projectiles moves away from the enemy in their heading direction towards the distance between startp and `playertargetentity`. The first one however specifically goes towards the `playertargetentity` position + 1.0 in y. Before each frame yield, the z angle of the projectly increased by 5 * the game's frametime
- DoDamage 1 call happens
- All spike projectiles are destroyed
- Yield all frames until `onground` goes to true
- animstate set to 105
- Yield for 0.8 seconds

## Move 2 - spin attack
A single target spin attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 3 times|This enemy|The selected `playertargetID`|1|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + 3.5 in x with 2.0 multiplier using the current animstate for start and stop
- Yield all frames until `forcemove` is done
- `Charge7` sound plays
- animstate set to 100
- Yield for 0.5 seconds
- `Woosh5` sound plays on loop with pitch 1.05
- animstate set to 103
- y `spin` set to 30.0
- Yield for 0.5 seconds
- Over the course of 50.0 frames (30.0 if hardmode is true), position is lerped to `playertargetentity` + (1.0, 0.0, -0.2)
- Done 3 times:
    - DoDamage 1 call happens
    - Yield for 0.5 seconds
- Over the course of 30.0 frames, position is lerped from the one it had before the previous lerp to `playertargetentity` + (3.0, 0.0, 0.0)
- `Woosh5` sound stopped
- `spin` zeroed out
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- animstate set to 105
- Yield for 0.8 seconds
