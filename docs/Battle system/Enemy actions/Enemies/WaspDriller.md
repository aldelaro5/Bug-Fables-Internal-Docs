# `WaspDriller`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This is set to 2 when using the digging underground move and every actor turn where this enemy attacks with [position](../../Actors%20states/BattlePosition.md) set to `Underground`, it is decremented. Once it reaches 0 after the decrement, the enemy goes back to `Ground`. Effectively, this value represents the amount of actor turns left that this enemy can stay `Underground` before going back to `Ground` by themselves.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the underground attack move, the total amount of frames for the enemy to move is 31.0 frames instead of being 39.0 frames. The [DoDamage](../../Damage%20pipeline/DoDamage.md) calls still happens on the first frame the enemy x position becomes lower than the `playertargetentity` one, but since the total amount of frames is reduced, this moment happens sooner
- In the grounded drill attack move, the enemy takes 26.0 frames to move to its target instead of 32.25 frames
- In the triple hits version of the grounded drill attack move, the yield time before the second and third hit changes to 0.45 seconds from 0.65 seconds
- In the rock debris attack move, the amount of rocks thrown is random between 2 and 4 inclusive instead of being random between 2 and 3 inclusive
- In the rock debris attack move, the speed of each projectile is random between 40 and 48 inclusive cast to float instead of being random between 40 and 56 cast to float

## Move selection
4 moves are possible:

1. A single target drill dash attack
2. A multiple targets rock debris thrown from drilling the ground
3. Digs to set the [position](../../Actors%20states/BattlePosition.md) to `Underground`
4. A single target underground drill attack

Move 4 is always (and only) used when [position](../../Actors%20states/BattlePosition.md) is `Underground`.

As for the other moves, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/8|
|2|3/8|
|3|2/8|

## Move 1 - Drill dash attack
A single target drill dash attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|`commandsuccess` is false (didn't blocked, ignores FRAMEONE)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>2</sup>|null|false|
|1_b|`commandsuccess` is true (blocked, ignores FRAMEONE), done 3 times|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|1|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more.

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md)
- Camera moves to look near `playertargetentity`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (7.0, 0.0, -0.1) with 2.0 multiplier using 1 (`Walk`) as walkstate and 5 (`Angry`) as stopstate
- Yield all frames until `forcemove` is done
- `Charge7` sound plays
- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 50.0 frametimer
- Yield for 0.75 seconds
- animstate set to 23 (`Chase`)
- `WaspDrill` sound plays on loop using `sounds[9]`
- Over the course of 32.25 frames (26.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (2.5, 0.0, -0.1) via a lerp
- If `commandsuccess` is false (the player didn't blocked, ignores FRAMEONE):
    - DoDamage 1_a call happens
    - Over the course of 61.0 frames, this enemy moves to (-20.0, 0.0, z doesn't change) via a lerp
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - `sounds[9]` stopped
    - Over the course of 71.0 frames, this enemy moves from (20.0, 0.0, z doesn't change) to startp via a lerp. Before each frame yield, animstate is set to 1 (`Walk`)
- Otherwise (the player blocked, ignores FRAMEONE):
    - `playertargetentity` animstate set to 11 (`Hurt`)
    - Done 3 times:
        - DoDamage 1_b call happens
        - Yield for 0.65 seconds (0.45 seconds instead if harmode is true)
    - Over the course of 31.0 frames, this enemy moves to + 2.0 in x via a lerp
    - `sounds[9]` stopped

## Move 2 - Rock debris throw
A multiple targets rock debris thrown from drilling the ground.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 to 3 times (2 to 4 times instead if hardmode is true), but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|null|This enemy|The return of [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) without nullable (target changes each calls)|A new sprite object rooted with a SpriteRenderer using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite positioned at this enemy|Random between 40 and 56 inclusive then cast to float (random between 40 and 48 inclusive instead if hardmode is true)|6.0|null|null|null|null|(0.0, 0.0, 20.0)|false|

### Logic sequence

- Camera moves to loop near (-2.45, 0.0, 1.9)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (0.5, 0.0, -0.5) with 2.0 multiplier using 1 (`Walk`) as walkstate and 5 (`Angry`) as stopstate
- Yield all frames until `forcemove` is done
- Yield for 0.5 seconds
- animstate set to 100
- `WaspDrill` sound plays on loop using `sounds[9]`
- `DirtFlying` particles plays near this enemy
- ShakeScreen called with an ammount of 0.05 and -1.0 time
- Yield for 0.5 seconds
- A new SpriteRenderer array is created with the amount of rocks to throw as the length. The amount is random between 2 and 3 inclusive (between 2 and 4 inclusive if hardmode is true)
- For each rock to throw, if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) without nullable is called
    - `DigPop` sound plays
    - The rock element is assigned to a new sprite object rooted with a SpriteRenderer using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite positioned at this enemy
    - Projectile 1 call happens
    - Yield for a random interval between 0.65 and 0.85 seconds
- Yield all frames until all rocks array elements are null (meaning all Projectile 1 calls completed)
- The `DirtFlying` particles created earlier are moved offscreen and destroyed in 5.0 seconds
- MainManager.`screenshake` zeroed out
- Yield for 0.3 seconds
- `sounds[9]` stopped

## Move 3 - Digging underground
Digs to set the [position](../../Actors%20states/BattlePosition.md) to `Underground`. No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect anything for this move.

### Logic sequence

- animstate set to 5 (`Angry`)
- Yield for 0.5 seconds
- `WaspDrill` sound plays on loop using `sounds[9]`
- animstate set to 100
- Yield for 0.35 seconds
- `data[0]` set to 2 (meaning this enemy will stay underground and attack for up to 2 actor turns)
- `checkingdead` set to a new [Dig](../Dig.md) coroutine for this enemy with 0 jump which eventually set [position](../../Actors%20states/BattlePosition.md) to `Underground` (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null
- `sounds[9]` stopped

## Move 4 - Underground drill attack
A single target underground drill attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md)
- Camera moves to look near `playertargetentity`
- `Digging` sound plays on loop using `sounds[9]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (4.0, 1.6, -0.1) with 1.6 multiplier
- Yield all frames until `forcemove` is done
- `digging` set to false
- animstate set to 101
- SetAnimForce called
- y position subtracted by 0.65 in y
- `sprite` has its z angle set to 35.0
- `sprite` has its scale set to Vector3.one
- `WaspDrill` sound plays on loop using `sounds[9]` with 1.3 pitch
- Yield for 0.5 seconds
- Over the course of 39.0 frames (31.0 frames instead if hardmode is true), this enemy moves -7.0 in x via a lerp. On the first frame the enemy x position becomes less than the `playertargetentity` x position, DoDamage 1 call happens
- Yield for 0.5 seconds
- `digging` set to true
- `sprite` has its scale set to Vector3.zero
- `sprite` has its angle reset to Vector3.zero
- y position set to 0.0
- `sounds[9]` stopped
- `data[0]` decremented
- `Digging` sound plays
- If `data[0]` reached 0 (meaning all underground actor turns were spent):
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with a 3.0 multiplier
    - Yield all frames until `forcemove` is done
    - `flip` set to false
    - `DirstExplode` particle plays at this enemy
    - `DigPop` sound plays
    - `digging` set to false
    - [position](../../Actors%20states/BattlePosition.md) set to `Ground`
    - Yield for 0.5 seconds
