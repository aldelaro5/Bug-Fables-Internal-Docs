# `Fisherman`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: An actor turn cooldown for the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) move. The value is set to 2 when the move is used and it is always decremented in the post move logic if it's still above 0. The delayed projectile move requires that the value is 0 or below for the move to be used. Effectively, it's an antispam of 1 full actor turn where the delayed projectile move can't be used

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the spear dash attack move, the odds to perform a loop before moving towards the target changes to 40% from 30%
- In the spear dash attack move, if a loop will be done before moving towards the target, it is now possible if a 50% RNG check passes for this eneny to jump twice instead of once at the start of the move
- In the spear dash attack move, if no loop will be done before moving towards the target, the amount of frames this enemy takes to move to its target changes to 14.0 frames from 19.0 frames
- In the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) move, the framespeed of the AddDelayedProjectile call changes to 35.0 from 48.0
- When using the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) move, the spear dash attack is now performed immediately after on the same actor turn

## Move selection
3 moves are possible:

1. A single target spear slash attack
2. A single target spear dash attack
3. Adds a [delayed projectile](../../Actors%20states/Delayed%20projectile.md) that can inflict a bad [condition](../../Actors%20states/Conditions.md) which may be followed up by performing move 1

If this enemy has the [Inked](../../Actors%20states/Conditions.md), move 1 is always used.

Otherwise, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/11|
|2|3/11|
|3|5|11|

However, if move 3 is selected and `data[0]` is above 0 (meaning to cooldown on it hasn't expired yet), move 2 is used instead.

## Post move logic
The following logic always happen after using a move:

- If `data[0]` is above 0, it is decremented.

## Move 1 - Spear slash attack
A single target spear slash attack.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|100|101|4|(2.0, 0.0, -0.1)|[Ink](../../Damage%20pipeline/AttackProperty.md)|0.0|Random interval between 0.5 and 0.65 seconds|`,RuffianKickSwing`|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (the BasicAttack 1 call completed)

## Move 2 - Spear dash attack
A single target spear dash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|[Ink](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 23 (`Chase`)
- The decision of whether or not to loop or not is determined. It will be done if a 30% RNG check passes (40% instead if hardmode is true)
- If we decided to loop, the amount of jumps to do is determined. It's 1, but if hardmode is true and a 50% RNG check passes, it's 2
- If we decided to loop, for each jump to do:
    - `Jump` sound plays
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy with an h of `jumpheight` / 1.5
    - Yield for 0.1 seconds
    - Yield all frames until this enemy is `onground`
- Otherwise (decided to not loop):
    - `Charge7` sound plays
    - ShakeSprite called with 0.3 intensity and 30.0 frametimer
    - Over the course of 31.0 frames, this enemy moves to + 2.0 in x via a lerp
- `trail` set to true
- If we decided to loop:
    - Over the course of 11.0 frames, this enemy moves to the result of a lerp from `playertargetentity` to startp with a factor of 0.75 (meaning 25% of the distance towards `playertargetentity`) via a lerp
- Otherwise (decided to not loop):
    - Camera moves to look near `playerdata[playertargetID]`
    - `Toss12` sound plays
    - Over the course of 19.0 frames (14.0 frames instead if hardmode is true), this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1) via a lerp
- If we decided to loop it is performed like the following:
    - `Spin3` sound plays with 0.9 pitch
    - Over the course of 31.0 frames:
        - This enemy x position goes from its existing one to BeizierFloat of -5.0 on the first half (will smoothly decrease up to 2.5 than back to no decrease) then on the second half, it will go from the existing one to BeizierFloat of 5.0 (will smoothly increase up to 2.5 than back to no increase). Basically, it will cycle to - 2.5 then back to the starting value halfway then + 2.5 then back to the starting value at the end
        - This enemy y position goes from its existing one to BeizierFloat of 10.0 (will smoothly increase to 5.0 than back to no increase)
        - This enemy z angle changes from 360.0 to 0.0 via a lerp
    - Camera moves to look near `playerdata[playertargetID]`
    - Over the course of 11.0 frames, this enemy moves to `playertargetentity` + (2.0, 0.0, -0.1) via a lerp
- DoDamage 1 call happens
- Over the course of 31.0 frames, this enemy moves to + 1.5 in x via a BeizierCurve3 with a ymax of 4.0
- `trail` set to false

## Move 3 - Adds a [delayed projectile](../../Actors%20states/Delayed%20projectile.md)
Adds a [delayed projectile](../../Actors%20states/Delayed%20projectile.md) that can inflict a bad [condition](../../Actors%20states/Conditions.md) which may be followed up by performing move 1. No damages are dealt for this move, but if move 1 is performed, that move will deal damages.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1<sup>*</sup>|Always happen|A new `Prefabs/Objects/BombCart` GameObject is instantiated childed to the `battlemap` at startp + Random.insideUnitCircle * 2.5 with a y of -2.0 with a scale of 0.0x with `Diggin` particles child playing infintely. The first child's `sprite` is an [item](../../../Enums%20and%20IDs/Items.md) sprite that depends on the property:<ul><li>null: `SpicyBomb`</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md): `SleepBomb`</li><li>[Numb](../../Damage%20pipeline/AttackProperty.md): `NumbBomb`</li><li>[Freeze](../../Damage%20pipeline/AttackProperty.md): `FrostBomb`</li><li>[Poison](../../Damage%20pipeline/AttackProperty.md): `PoisonBomb`</li><li>[InkOnBlock](../../Damage%20pipeline/AttackProperty.md): `HoneyDrop` with a color of 400040 (dark purple)</li></ul>|`playertargetID`|Depends on the property:<ul><li>3 if property is null</li><li>2 otherwise</li></ul>|2|0|Determined randomly with the following odds:<ul><li>1/7: null</li><li>1/7: [Sleep](../../Damage%20pipeline/AttackProperty.md)</li><li>1/7: [Numb](../../Damage%20pipeline/AttackProperty.md)</li><li>1/7: [Freeze](../../Damage%20pipeline/AttackProperty.md)</li><li>1/7: [Poison](../../Damage%20pipeline/AttackProperty.md)</li><li>2/7: [InkOnBlock](../../Damage%20pipeline/AttackProperty.md)</li></ul>|48.0 (35.0 instead if hardmode is true)|This enemy|Depends on the property:<ul><li>null: `Explosion`</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md): `Charge`</li><li>[Numb](../../Damage%20pipeline/AttackProperty.md): `Charge19`</li><li>[Freeze](../../Damage%20pipeline/AttackProperty.md): `IceMelt`</li><li>[Poison](../../Damage%20pipeline/AttackProperty.md): `ChargeDown2`</li><li>[InkOnBlock](../../Damage%20pipeline/AttackProperty.md): `WaterSplash2`</li></ul>|Depends on the property:<ul><li>null: `explosionsmall`</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md): `deathsmoke`</li><li>[Numb](../../Damage%20pipeline/AttackProperty.md): `ElecFast`</li><li>[Freeze](../../Damage%20pipeline/AttackProperty.md): `mothicenormal`</li><li>[Poison](../../Damage%20pipeline/AttackProperty.md): `PoisonEffect`</li><li>[InkOnBlock](../../Damage%20pipeline/AttackProperty.md): `InkGet`</li></ul>|`Digging@Down`|

*: The `delproj` gets the following [args](../../Actors%20states/Delayed%20projectile.md#args-field): `move,0,-0.5,0@noshadow@partoff,0,0.5,0@partoff,0,1,0`

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp + Random.insideUnitCircle * 2.5 with a y component of 0.0 using 2.0 multiplier
- Camera moves to look near the point this enemy is moving towards
- Yield all frames until `forcemove` is done
- A new `Prefabs/Objects/BombCart` GameObject is instantiated childed to the `battlemap` at this enemy position - 2.0 in y
- animstate set to 102
- The effects of the delayed projectile is determined here (see the AddDelayedProjectile table above for details)
- If the effect of the delayed projectile won't be with `InkOnBlock` property, the `Prefabs/Objects/BombCart`'s first child's `sprite` is set to the corresponding [item](../../../Enums%20and%20IDs/Items.md) sprite of the effect
- Otherwise, the `Prefabs/Objects/BombCart`'s first child's `sprite` is set to a `HoneyDrop` [item](../../../Enums%20and%20IDs/Items.md) sprite with a color of 400040 (dark purple)
- Yield for 0.5 seconds
- `Dig2` sound plays with 1.1 pitch
- Yield for 31.0 frames counted locally
- `Digging` particles plays at this enemy position + (-0.25, 0.0, -0.1) with -1.0 alivetime (infinite)
- Yield for 0.5 seconds
- Over the course of 11.0 frames, `Prefabs/Objects/BombCart`'s scale changes to Vector3.zero via a lerp
- The `Digging` particles gets childed to `Prefabs/Objects/BombCart`
- The `Digging` particles has their scale set to 1.0x
- AddDelayedProjectile 1 call happens
- The `delproj` that was just added has its `args` set to `move,0,-0.5,0@noshadow@partoff,0,0.5,0@partoff,0,1,0`
- animstate set to 103
- Yield for 0.5 seconds
- `data[0]` is set to 2 (meaning this move can't be used on the next actor turn, but it will become usable in 2 actor turns)
- If hardmode is true, the spear dash attack move is performed immediately without ending the actor turn
