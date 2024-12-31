# `Ahoneynation`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: UNUSED

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the honey throw move, the amount of frames each projectile takes to reach its target changes to 21.0 frames from 23.5 frames

## Move selection
4 moves are possible:

1. A single target punch attack
2. A single target honey throw that may hit multiple times and summon new [Abomihoney](Abomihoney.md) enemies
3. Prepares themselves for a big attack by setting `charge` to 2
4. A party wide jump attack

Move 4 is always used (and only used) if `basestate` isn't 0 (`Idle`) meaning move 3 was performed last actor turn.

As for other moves, the decision of which moves to use is based on odds, but the odds changes when [HPPercent](../../Actors%20states/HPPercent.md) is 0.6 or less. Move 2 and 3 have additional requirements that if failed when selecting the move, move 1 will be performed instead. Here are the odds and requirements:

|Move|Odds when [HPPercent](../../Actors%20states/HPPercent.md) floored > 0.6|Odds when [HPPercent](../../Actors%20states/HPPercent.md) <= 0.6|Additional requirments|
|---:|----|----|----|
|1|1/4|1/6|None, the move is always used when selected|
|2|2/4|2/6|The enemy party must have less than 3 enemy party members|
|3|1/4|3/6|[HPPercent](../../Actors%20states/HPPercent.md) must be between 0.15 exclusive and 0.65 exclusive (between 15% exclusive and 65% exclusive `hp` remaining)|

## Move 1 - Punch attack
A single target punch attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|5|null|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentity`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (4.85, 0.0, -0.1) with a 0.7 multiplier
- Yield until `forcemove` is done and this enemy is `onground`
- animstate set to 100
- `Charge18` sound plays
- Yield for 0.75 seconds
- animstate set to 102
- Yield for 0.06 seconds
- ShakeScreen called with 0.3 ammount and 0.5 time
- DoDamage 1 call happens
- `BigHit` sound plays
- `impactsmoke` particles plays at `playertargetentity` position
- Yield for 0.75 seconds

## Move 2 - Honey throw
A single target honey throw that may hit multiple times and summon new [Abomihoney](Abomihoney.md) enemies.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen from 1 to 2 times (always 1 time if there's already 2 or more enemy party members). Each call requires that at least one player party member is alive (`hp` above 0 and not [EatenBy](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|3|[Numb](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- Camera moves a bit forward
- `AhoneynationSpitCharge` sound plays
- animstate set to 104
- Yield for 0.65 seconds
- The amount of hits is determined. It's random between 1 and 2 inclusive (always 1 instead if there's already 2 or more enemy party members)
- For each hit:
    - If no player party members are alive (`hp` above 0 and not [EatenBy](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)), this hit and all further ones are skipped
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - animstate set to 106
    - `AhoneynationSpit` sound plays
    - [CreateNewEntity](../../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) called with name `enemy`, [animid](../../../Enums%20and%20IDs/AnimIDs.md) of 177 (`Abomihoney`) positioned offscreen at (0.0, 999.0, 0.0)
    - `enemy`'s `hologram` gets set to this enemy's value
    - `enemy`'s `battle` gets set to true (this is needed because if they are summoned, the search for the free spot filters based on this)
    - Yield a frame
    - `enemy`'s `startscale` set to 0.75x
    - `enemy`'s `shadowsize` set to 1.1
    - `enemy`'s animstate set to 108
    - Yield for 0.1 seconds
    - animstate set to 107
    - Over the course of 23.5 frames (21.0 frames instead if hardmode is true), `enemy` moves to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp
    - DoDamage 1 call happens
    - `ElecFast` particles plays at `playertargetentity` position + (0.0, 1.0, -0.1)
    - If `commandsuccess` is true (blocked, ignores FRAMEONE), the enemy will be slated for summoning by doing the following:
        - The location of the enemy is determined. The locations considered in order are (0.5, 0.0, 0.85) and (6.55, 0.0, -0.5). The first free one (no `battle` EntityControl within a 1.0 radius) will be selected, but it defaults to (-15.0, 5.0, 0.0) if neither are free the enemy won't be summoned (this can happen if other [Abomihoneis](Abomihoney.md) called [CleanKill](../../Actors%20states/CleanKill.md) since they never got destroyed even if they got removed from `enemydata`)
        - `enemy` moves to the location - 0.2 in z via ArcMovement without spin and with a height of the distance of `enemy` to the location clamped from 2.0 to 4.5 for 35.0 frametime without destroyonend
- animstate set to 0 (`Idle`)
- Yield for 0.75 seconds if any enemy are slated for summoning through a projectile entity. For each enemy to summon:
    - animstate set to 107
    - Yield for 0.4 seconds
    - `Puddle` sound plays
    - [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add an [Abomihoney](Abomihoney.md) offscreen at (0.0, 999.0, 0.0)
    - Yield for a frame
    - `enemydata[lastaddedid]` is positioned at the matching projectile entity position
    - The projectile entity gets destroyed
    - The new enemy gets some fields changed:
        - animstate and `basestate`: 0 (`Idle`)
        - `startscale`: 0.75x
        - `hp` and `maxhp`: divided by 3 floored
        - `def`: decremented. NOTE: This can incorrectly result in a value of -1 which can impact the damage pipeline in unexpected ways (such as increasing the result by 1) and to cause [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#truedef) to incorrectly report a `?` defense value when this enemy won't have a `LimitX10` [weakness](../../Actors%20states/Enemy%20features.md#weakness)
        - `exp`: 1 (0 instead if the `Ahoneynation`'s `hologram` is true)
        - `locktri`: true
        - `cantmove`: 1 (meaning this enemy cannot act on this main turn and needs to wait the current main turn advances)

## Move 3 - Charging
Braces themselves to set `charge` to 2. No damages are dealt. The `basestate` changes allowing move 4 (party wide jump attack) to be used on the next actor turn.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- animstate set to 109
- `Charge18` sound plays with 0.9 pitch
- animstate set to 109
- Over the course of 61.0 frames, y `spin` goes to 15.0 via a lerp
- Yield for 0.3 seconds
- animstate, `basestate` and the local startstate set to 108
- `spin` zeroed out
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- `charge` set to 2
- Yield for 0.5 seconds

## Move 4 - Party wide jump attack
A party wide jump attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `AhoneynationBodySlamCharge` sound plays
- Camera moved to look near (-4.0, 1.0, 0.0)
- Over the course of 51.0 frames, the `rotater`'s SpriteBounce (this is setup by [CheckSpecialID](../../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md)) has their `basescale` changed to (1.2, 0.5, 1.0) via a lerp and their `basespeed` changed to be multiplied by 1.5 via a lerp
- Yield for 0.1 seconds
- y `spin` set to 20.0
- animstate set to 109
- `rotater`'s SpriteBounce's `basescale` reset to its value before this action
- `AhoneynationBodySlamJump` sound plays
- Over the course of 61.0 frames, this enemy moves to `partymiddle` via a BeizierCurve3 with a ymax of 15.0
- position set to `partymiddle`
- `spin` zeroed out
- animstate set to 108
- `impactsmoke` particles played at this enemy position
- ShakeScreen called with 0.3 ammount and 0.5 time
- PartyDamage 1 call happens
- `Thud3` sound plays
- Over the course of 61.0 frames, `rotater`'s SpriteBounce's `frequency` goes from 1.0 to 0.0 via a lerp
- `rotater`'s SpriteBounce's `basescale` reset to its value before this action
- `rotater`'s SpriteBounce is SetUp with the frequency and speed it had before this action
- animstate, `basestate` and the local startstate set to 0 (`Idle`)
