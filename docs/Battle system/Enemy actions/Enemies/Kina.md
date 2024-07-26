# `Kina`

## Assumptions
This enemy is assumed to be fought with [Maki](Maki.md) for the move selection to function otherwise, unexpected logic will occur.

It is also assumed that the [DeathType](../../Actors%20states/Enemy%20features.md#deathtype) of the enemy supports the `reservedata` feature as it is needed by [Maki](Maki.md) in case they want to revive this enemy.

## [StartBattle](../../StartBattle.md) special logic
After loading this enemy, the following special adjustements happens to them if [flags](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active):

- `maxhp` and `hp` gets increased by 10
- `def` gets incremented

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This tells if this enemy should receive an infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition. The value becomes 1 on the first actor turn's pre move logic or `hitaction` that [Maki](Maki.md) isn't found in `enemydata`. When the value is 1 at the end of the pre move logic, it allows this enemy to receive an infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition if they didn't had it already

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the projectiles throw move, the odds that the property of each Projectile calls is [Poison](../../Damage%20pipeline/AttackProperty.md), [Sleep](../../Damage%20pipeline/AttackProperty.md) or [Numb](../../Damage%20pipeline/AttackProperty.md) changes to 50% from 70%
- In the projectiles throw move, the speed of each projectile call changes to 12.0 from 19.0
- In the jump kick attack move, the amount of frames this enemy takes to reach its target changes to 21.0 from 28.0. However, after this, there is always a yield of 0.1 seconds regardless if hardmode is true or not before the DoDamage call

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### `dontusecharge` set to true
The `hitaction` logic always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- The `enemydata` index of [Maki](Maki.md) is obtained if they are present
- If `Maki` is present in `enemydata`:
    - `Wam` sound plays
    - [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote for 45 time
    - animstate set to 5 (`Angry`)
    - Yield for 0.5 seconds
    - Yield for 0.25 seconds
    - If `charge` is less than 3:
        - `charge` is incremented
        - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
        - `StatUp` sound plays
        - Yield for 0.5 seconds
- Otherwise (`Maki` isn't present), if `data[0]` is 0 (meaning their absence wasn't noticed before):
    - animstate set to 5 (`Angry`)
    - Done 2 times:
        - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
        - `Jump` sound plays with 1.2 pitch
        - Yield for 0.25 seconds
        - Yield all frames until this enemy is `onground`
    - `data[0]` is set to 1 (indicating that it was found that `Maki` died which means on the next actor turn, this enemy will get an infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition in the pre move logic)

## Move selection
3 moves are possible:

1. A multiple targets projectiles throw with varying effects
2. A single target jump kick attack
3. Heals [Maki](Maki.md) by 6 `hp` and inflicts them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/9|
|2|4/9|
|3|2/9|

However, if move 3 is selected and [Maki](Maki.md) is either not present or their [HPPercent](../../Actors%20states/HPPercent.md) is above 0.7, a continue directive is issued to the enemy action loop which restarts the entire action. Effectively, this will reroll the move selection process.

## Pre move logic
The following logic always happen before a move is used:

- The `enemydata` index of [Maki](Maki.md) is obtained if they are present
- If `Maki` isn't present and `data[0]` is 0 (meaning their absence wasn't noticed before):
    - animstate set to 5 (`Angry`)
    - Done 2 times:
        - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
        - `Jump` sound plays with 1.2 pitch
        - Yield for 0.25 seconds
        - Yield all frames until this enemy is `onground`
    - `data[0]` is set to 1 (indicating that it was found that `Maki` died)
- If `data[0]` is 1 (it was found that `Maki` died just now or in a previous `hitaction`) and this enemy doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition, an infinite one will be inflicted with the following:
    - `Charge7` sound plays
    - ShakeSprite called with 0.1 intensity and 45.0 frametimer
    - Yield for 0.5 seconds
    - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 999999 main turns (infinite) with effect
    - Yield for 0.5 seconds

## Move 1 - Projectiles throw
A multiple targets projectiles throw with varying effects.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 to 4 times determined randomly, but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|null, but there is a 70% chance (50% instead if hardmode is true) that the property is determined randomly among the following with uniform odds:<ul><li>[Poison](../../Damage%20pipeline/AttackProperty.md)</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md)</li><li>[Numb](../../Damage%20pipeline/AttackProperty.md)</li></ul>|This enemy|`playertargetID`|A new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) positioned at this enemy + (-1.5, 1.5, -0.1) with a scale of 0.6x and a z angle of -90.0. The color depends on the property:<ul><li>[Poison](../../Damage%20pipeline/AttackProperty.md): pure magenta</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md): pure green</li><li>[Numb](../../Damage%20pipeline/AttackProperty.md): pure yellow</li></ul>|19.0 (12.0 instead if hardmode is true)|0.0|`keepcolor,x`|null|null|null|Vector3.zero|false|

### Logic sequence

- The property of each Projectile 1 call is determined here (see the Projectile calls table above for details)
- The amount of hits is determined which is random from 2 to 4 inclusive. A SpriteRenderer array is created with this amount as the length to hold the projectiles objects
- For each hits to do, if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - Camera moves to look near the midpoint between this enemy and `playertargetentity`
    - animstate set to 102
    - Yield for 0.5 seconds
    - `FastWoosh` sound plays
    - animstate set to 103
    - The projectile element is set to a new sprite object childed to the `battlemap` using the `projectilepsrites[12]` sprite (purple pointy projectile) positioned at this enemy + (-1.5, 1.5, -0.1) with a scale of 0.6x and a z angle of -90.0
    - The color of the projectile is set which depends on the property determined earlier:
        - [Poison](../../Damage%20pipeline/AttackProperty.md): pure magenta
        - [Sleep](../../Damage%20pipeline/AttackProperty.md): pure green
        - [Numb](../../Damage%20pipeline/AttackProperty.md): pure yellow
    - Projectile 1 call happens
    - Yield for 0.5 seconds
- animstate set to 13 (`BattleIdle`)
- Yield all frames until all projectiles elements are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 2 - Jump kick attack
A single target jump kick attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|5|null|null|`commandsuccess`|

### Logic sequence

- animstate set to 103
- `Toss3` sound plays
- Yield for 0.5 seconds
- `PingShot` sound plays
- DeathSmoke particles plays at this enemy position + 0.5 in y with a size of 3.0x
- Yield for a frame
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- position set to `playertargetentity` + (0.0, 15.0, -0.1)
- `trail` set to true
- animstate set to 104
- Yield for 0.5 seconds
- Camera moves to look near `playerdata[playertargetID]`
- Yield for 0.25 seconds
- `Toss12` sound plays
- Over the course of 28.0 frames (21.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` + (0.0, 1.5, -0.1) via a lerp
- animstate set to 105
- Yield for 0.1 seconds
- `Woosh5` sound plays with 0.8 pitch
- DoDamage 1 call happens
- animstate set to 101
- SetAnimForce called on this enemy
- z `spin` set to -25.0
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Yield for a frame
- ShakeScreen called with an ammount of 0.2 and 0.75 time with dontreset
- Over the course of 61.0 frames, this enemy moves from `playertargetentity` + (0.0, 2.5, -0.1) to startp + 1.0 in y via a BeizierCurve3 with a ymax of 7.0
- `Drop` sound plays
- `trail` set to false
- `spin` zeroed out
- animstate set to 106
- `sprite` angles reset to Vector3.zero
- position set to startp
- SetAnimForce called on this enemy
- Yield for 0.25 seconds

## Move 3 - Heals [Maki](Maki.md) with [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) infliction
Heals [Maki](Maki.md) by 6 `hp` and inflicts them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition. No damages are dealt.

### Logic sequence

- animstate set to 4 (`ItemGet`)
- `ItemHold` sound plays
- A new sprite object is created childed to the `battlemap` using the `MiteBurg` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + (-0.43, 3.9, -0.1). The object is destroyed in 1 second
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called to heal `Maki` by 6 `hp`
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition on `Maki` for 3 main turns
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
