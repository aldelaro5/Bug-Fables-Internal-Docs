# `MotherChomper`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In both projectile throw moves, the amount of hits is random from 2 to 3 inclusive instead of being random between 1 to 3 inclusive
- In both projectile throw moves, the amount of frames the projectile takes to reach its target changes to 35.0 frames from 41.0

## Move selection
3 moves are possible:

1. A multiple targets seeds projectile throw
2. A multiple targets spiky projectiles throw that can infllict the [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition
3. A bite attack to the player party member in front

The move usage decision is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/7|
|2|2/7|
|3|2/7|

However, move 3 has additional requirements that must be fufilled after being selected for the move to be used:

- `playerdata[partypointer[0]]` (the front player party member) must have their `hp` above 0
- Either `forceattack` is -1 (the player didn't taunted the enemy party) or it is the same as `playerdata[partypointer[0]]` (the front player party member is taunting the enemy party)

If these requirements aren't fufilled and the move is selected, move 2 is used instead.

## Pre move logic
There are 2 parts of logic that always happen at the start of the actor turn if their conditions are fufilled. They are mentioned in order they appear.

### [FlyTrap](FlyTrap.md) summon
If this enemy is the last enemy party member remaining, the following happens:

- animstate set to 104
- Yield for 0.75 seconds
- animstate set to 5 (`Angry`)
- `Roar` sound plays with 1.1 pitch
- ShakeScreen called with 0.2 ammount and 0.75 time
- Yield for 0.75 seconds
- animstate set to 101
- `Clomp` sound plays
- Camera moves a bit up and right
- Yield for 1 second
- animstate set to 108
- A new sprite object is created rooted using the `ChomperSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite position at this enemy position + (-0.75, 7.6, 0.1) with a SpinAround that has an `itself` of 20.0 in z
- Over the course of 120.0 frames, the `ChomperSeed` sprite moves to (1.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 9.0. Before each frame yield after the halfway mark, the animstate of this enemy is set to 0 (`Idle`)
- DeathSmoke particles plays at the `ChomperSeed` sprite position
- `Charge` sound plays
- `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new [FlyTrap](FlyTrap.md) enemy with type `FromGroundKeepScale` at the `ChomperSeed` position with cantmove (meaning the new enemy cannot act on this main turn and needs to wait on the next one)
- The `ChomperSeed` sprite gets destroyed
- Yield for a frame
- Yield all frames until `checkingdead` is null (the coroutine completed)

### Phase transition
If [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.6 and `moves` is still 1 (meaning the phase transition hasn't happened yet), the following phase transition occurs which notably sets `move` to 2 granting 2 actor turns per main turn to this enemy (excluding the current main turn):

- animstate set to 5 (`Angry`)
- `Roat` sound plays with 1.1 pitch
- ShakeScreen called with 0.2 ammount and 0.75 time
- Yield for 0.75 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
- Yield for 0.75 seconds
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
- `moves` set to 2

## Move 1 - Seeds throw
A multiple targets seeds projectile throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen from 1 to 3 times inclusive (from 2 to 3 times inclusive instead if hardmode is true). Each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1, but this will be guaranteed to be the case if at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup> (target changes for each calls)|3|`None` (same as null)|null|`commandsuccess`|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- Camera moves to look at (0.7, 0.3, 1.0)
- animstate set to 101
- `Clomp` sound plays
- Yield for 1.25 seconds
- A new sprite object is created rooted using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned offscreen at 999.0 in y with a SpinAround that has an `itself` of 20.0 in z
- The amount of hits is determined. It's random between 1 and 3 inclusive (between 2 and 3 inclusive instead if hardmode is true)
- For each hit:
    - `playertargetID` is set to [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable
    - If the above returned -1 (couldn't find a target), the hit is rerolled without counting it as long as not every player party members are dead (`hp` at 0 or below or [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) followed by a frame yield
    - Otheriwse (a target was found):
        - `playertargetentity` is set to the matching `playertargetID`'s `battleentity`
        - The `HardSeed` sprite position is set to this enemy position + (-5.0, 2.25, -0.1)
        - animstate set to 103
        - `Spit` sound plays
        - Over the course of 41.0 frames (35.0 frames instead if hardmode is true), the `HardSeed` sprite moves to `playertargetentity` position + (0.0, 1.0, -0.1.0) via a lerp. Before each frame yield before the halfway mark, if this isn't the last hit, this enemy animstate is set to 101
        - The `HardSeed` sprite is moved offscreen at (0.0, 999.0, 0.0)
        - DoDamage 1 call happens
- The `HardSeed` sprite gets destroyed
- Yield for 0.5 seconds

## Move 2 - Spiky projectiles throw
A multiple targets spiky projectiles throw that can infllict the [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen from 1 to 3 times inclusive (from 2 to 3 times inclusive instead if hardmode is true). Each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1, but this will be guaranteed to be the case if at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup> (target changes for each calls)|2|[Sleep](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence
This move has the same logic secquence than move 1 (seeds throw) with 3 changes:

- DoDamage 1 deals 2 instead of 3 as outlined above
- DoDamage has the `Sleep` property instead of `None` as outline above
- The sprite used as the projectile is `projectilepsrites[5]` (a gray spiky projectile) instead of the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite

## Move 3 - Bite
A bite attack to the player party member in front.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playerdata[partypointer[0]]` (the player party member in front)|5|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- `Chew` sound plays with 0.85 pitch
- Camera moves to look near (1.0, 0.0, 1.76)
- `playertargetID` is set to `partypointer[0]` (the player party member in front)
- `sprite` y angle set to -1.0
- animstate set to 104
- Yield for 1.1 seconds
- animstate set to 106
- Camera moves to look near (-2.0, -1.0, 3.0)
- `Bite` sound plays with 0.9 pitch
- Yield for 0.2 seconds
- DoDamage 1 call happens
- Yield for 0.5 seconds
