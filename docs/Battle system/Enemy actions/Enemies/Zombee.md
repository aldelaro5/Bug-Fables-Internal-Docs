# `Zombee`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The move usage odds changes to favor more the darts throws move
- In the darts throws move, the speed of each projectile changes to 21.0 from 32.0

## Move selection
2 moves are possible:

1. A single target double slash
2. A multiple targets triple darts throws

The decision of which move to use are based on the following odds which changes based on if hardmode is true or not:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|---------------------------|--------------------------|
|1|59%|39%|
|2|41%|61%|

## Move 1 - Double slash
A single target double slash.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|100|102|2|(1.25, 0.0, -0.1)|[Poison](../../Damage%20pipeline/AttackProperty.md)|0.0|0.5 seconds|`,Toss7`|false|
|2|Always happen|This enemy|1 (`Walk`)|102|103|2|(1.25, 0.0, -0.1)|[Sleep](../../Damage%20pipeline/AttackProperty.md)|0.0|0.2 seconds|`,Toss8`|true|

### Logic sequence

- `checkingdead` set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (BasicAttack 1 completed)
- `checkingdead` set to the BasicAttack 2 call
- Yield all frames until `checkingdead` is null (BasicAttack 2 completed)

## Move 2 - Triple darts throws
A multiple targets triple darts throws.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 3 times, but each calls requires to have at least 1 player party member whose `hp` is above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)|3|[Sleep](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new sprite object using the `projectilepsrites[2]` sprite (spiky projectile) positioned at this enemy + (-1.0, 1.0, -0.1) with a pure gray color and a z angle of -90.0|32.0 (21.0 instead if hardmode is true)|0.0|null|null|null|null|Vector3.zero|false|

### Logic sequence

- Done 3 times:
    - If there isn't at least 1 player party member whose `hp` is above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences), this action ends prematurely
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `Rope` sound plays
    - Camera moves to look near the midpoint between this enemy and `playertargetentity`
    - animstate set to 28 (`TossItem`)
    - Yield for 0.5 seconds
    - `Toss` sound plays
    - A new sprite object is created using the `projectilepsrites[2]` sprite (spiky projectile) positioned at this enemy + (-1.0, 1.0, -0.1) with a pure gray color and a z angle of -90.0
    - DoProjectile 1 call happens
    - Yield for 0.33 seconds
    - animstate set to 0 (`Idle`)
    - Yield all frames until the sprite object is null (Projectile 1 completed)
    - Yield for 0.5 seconds
