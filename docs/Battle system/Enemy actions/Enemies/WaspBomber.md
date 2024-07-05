# `WaspBomber`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 1 change:

- In the bomb throw move, the amount of frames the bomb takes to reach its target is random between 41 and 50 frames instead of being random between 41 and 60 frames

## Move selection
1 moves is possible:

1. A party wide bomb throw of a random type

Move 1 is always used.

## Move 1 - Bomb throw
A party wide bomb throw of a random type.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md) calls

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2 (4 instead if the bomb thrown is a `SpicyBomb`)|Depends on the bomb thrown:<ul><li>`SpicyBomb` or `BurlyBomb`<sup>1</sup>: null</li><li>`NumbBomb`: [Numb](../../Damage%20pipeline/AttackProperty.md)</li><li>`FrostBomb`: [Freeze](../../Damage%20pipeline/AttackProperty.md)</li><li>`SleepBomb`: [Sleep](../../Damage%20pipeline/AttackProperty.md)</li><li>`PoisonBomb`: [Poison](../../Damage%20pipeline/AttackProperty.md)</li></ul>|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence
The bomb type is determined with random odds:

- `SpicyBomb`: 3/7
- `PoisonBomb`: 1/7
- `SleepBomb`: 1/7
- `FrostBomb`: 1/7
- `NumbBomb`: 1/7

However, if it's a `SleepBomb`, `FrostBomb` or `NumbBomb` and any of the player party members have the [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) conditions, the result is rerolled to be one of the following with equal chances each:

- `SpicyBomb`
- `PoisonBomb`
- `BurlyBomb`

After determining the bomb type, this is what the logic does:

- animstate set to 100
- `TwistDoorEnter` sound plays
- A new sprite object is created rooted with the matching [item](../../../Enums%20and%20IDs/Items.md) sprite at this enemy position + (-0.6, 1.4, -0.1)
- Over the course of 23.0 frames, the bomb moves to this enemy position + (-0.6, 3.2, -0.1)
- Yield for 0.15 seconds
- `Toss12` sound plays
- Over the course of an amount of frames between 41.0 and 60.0 (the upper bound is 50.0 instead), the bomb moves to `partymiddle` via a BeizierCurve3 with a ymax of the total amount of frames / 10.0 + 2. Before each frame yield, the z angle of the bomb increases by 20.0
- `Explosion` sound plays
- Different effects happens depending on the bomb type (except for `BurlyBomb` where no effects happen):
    - `SpicyBomb`:
        - ShakeScreen called with an ammount of 0.2 and 0.75 time with dontreset
        - `Explosion` particles plays at `partmymiddle`
    - `PoisonBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `PoisonEffect` particles plays at `partmymiddle` with 2x scale
    - `SleepBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - DeathSmoke particles plays at `partmymiddle` with 2x size
    - `FrostBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `mothicenormal` particles plays at `partmymiddle` with 3x scale
    - `NumbBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `ElecFast` particles plays at `partmymiddle` with 2x scale
        - `impactsmoke` particles plays at `partmymiddle`
- PartyDamage 1 call happens
- The bomb gets destroyed
- Yield for 0.5 seconds
