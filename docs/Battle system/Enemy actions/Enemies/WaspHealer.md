# `WaspHealer`

## [GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy features special exp handling logic. Check the documentation to learn more.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 5 changes:

- In the needles throw move, the property of all the [Projectile](../../Damage%20pipeline/Projectile.md) calls is always [Poison](../../Damage%20pipeline/AttackProperty.md) instead of only being `Poison` with a 6/10 chance and null with a 4/10 chance
- In the needles throw move, the damage is always 1 instead of being 2 (3 instead if instance.`areaid` is the `RubberPrison` or `MetalLake` [areas](../../../Enums%20and%20IDs/librarystuff/Areas.md))
- In the needles throw move, the speed is 10 instead of 15
- In the needles throw move, the yield time between each hits is 0.35 seconds instead of being 0.25 seconds
- In the aerial strike move, the amount of frames this enemy takes to reach its target from near the target is 9.0 frames instead of 11.0 frames

## Move selection
1 moves are possible:

1. A single target aerial needles throw that hits twice
2. A single target aerial strike
3. Summons another `WaspHealer` enemy and change [position](../../Actors%20states/BattlePosition.md) to `Ground`.

Here are the base odds to use each move:

|Move|Odds|
|---:|----|
|1|2/4|
|2|1/4|
|3|1/4|

However, in order for move 3 to actually be used, the following conditions needs to be fufilled after selecting it:

- This enemy is alone in `enemydata`
- `locktri` is false (meaning it hasn't summoned before and it wasn't a summoned `WaspHealer`)

If the move was selected, but the conditions above aren't fufilled, move 2 is used instead.

## Move 1 - Needles throw
A single target aerial needles throw that hits twice.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 times|<ul><li>If harmode is false, it's 2 (3 instead if instance.`areaid` is the `RubberPrison` or `MetalLake` [areas](../../../Enums%20and%20IDs/librarystuff/Areas.md))</li><li>If hardmode is true, it's always 1</li></ul>|null ([Poison](../../Damage%20pipeline/AttackProperty.md) instead if hardmode is true)|This enemy|`playertargetID`|A new GameObject childed to `battlemap` with a SpriteRenderer using the `projectilepsrites[2]` sprite (a needle) with the `spritemate` material (if hardmode is true color is set to pure magenta) positioned at this enemy's `sprite` + (-1.5, 0.25, -0.1) with a z angle of -45.0 on layer 14 (`Sprite`)|15 (10 if hardmode is true)|0.0|null|null|null|null|Vector3.zero|false|

### Logic sequence

- `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call on this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` is set to null when completed)
- Yield until `checkingdead` is null
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.25, 0.0, -0.25) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- Yield for 0.4 seconds
- A new Transform array of 2 elements is created to contain each needles
- Done 2 times for each needle to throw:
    - `Toss` sound plays
    - animstate set to 102
    - Needle element initialized to a new GameObject childed to `battlemap` with a SpriteRenderer using the `projectilepsrites[2]` sprite (a needle) with the `spritemate` material (if hardmode is true color is set to pure magenta) positioned at this enemy's `sprite` + (-1.5, 0.25, -0.1) with a z angle of -45.0 on layer 14 (`Sprite`)
    - Projectile 1 call happens
    - Yield for 0.25 seconds (0.35 instead if hardmode is true)
    - If this is the last needle:
        - animstate set to 100
        - Yield for 0.4 seconds
- Yield all frames until all the needles array elements are null (meaning all Projectile calls completed)
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds

## Move 2 - Aerial strike
A single target aerial strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3 (4 instead if instance.`areaid` is the `RubberPrison` or `MetalLake` [areas](../../../Enums%20and%20IDs/librarystuff/Areas.md))|null|null|`commandsuccess`|

### Logic sequence

- `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call on this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` is set to null when completed)
- Yield until `checkingdead` is null
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.25, 0.0, -0.25) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 101
- `height` set to 0.0
- y position increased by the `height` value before this action
- `BugWingFast` sound plays
- Over the course of 31.0 frames, this enemy moves +1.25 in x via a BeizierCurve3 with a ymax of -1.5
- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 105
- `trail` set to true
- `Woosh2` sound plays
- Over the course of 11.0 frames (8.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (0.75, 0.75, -0.1) via a lerp
- ShakeScreen called with an ammount of 0.05 and time of 0.2
- DoDamage 1 call happens
- Yield for 0.35 seconds
- animstate set to 104
- Over the course of 11.0 frames, this enemy moves to `playertargetentity` position + (2.0, `height` value of this enemy before this action, -0.2) via a lerp
- `trail` set to false
- `height` set to the value before this action
- y position set to 0.0

## Move 3 - Summons another `WaspHealer`
Summons another `WaspHealer` enemy and change [position](../../Actors%20states/BattlePosition.md) to `Ground`. No damages are dealt.

### nocharm set to true
This move always set the nocharm local to true which will prevent any [UseCharm](../../Battle%20flow/UseCharm.md) calls to occur in the post action phase. This means no `HealHP` charm can occur after this action.

### Logic sequence

- `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call on this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Ground` (`checkingdead` is set to null when completed)
- Yield until `checkingdead` is null
- animstate set to 106
- Yield for 0.3 seconds
- `Whistle` sound plays with 0.9 pitch and 0.7 volume
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new `WaspHealer` with an `Offscreen` type at a random location whose between (0.25, 0.0, -0.75) and (7.0, 0.0, 0.75) where the distance between any enemy party member is less than its `size` * 1.75 with cantmove (meaning the enemy needs to wait on the next main turn to act)
- Yield for 0.85 seconds
- animstate set to 0 (`Idle`)
- `locktri` set to true (prevents from using this move again)
- `enemydata[lastaddedid]`'s `locktri` set to true which prevents the summoned enemy from using this move. NOTE: It is technically possible that the summoning hasn't completed yet, but the previous yield makes it very unlikely in practice
- Yield until `checkingdead` is null (the coroutine completed)
