# `Stratos`

## Assumptions
It is assumed this enemy is fought with them and [Delilah](Delilah.md) only in the enemy party as they strongly interact together. If this assumption isn't respected, exceptions may be thrown.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 5 element with a starting value of 0.

- `data[0]`: The amount of times the reviving move or the enemy party healing move are used. The value is incremented each time either of these moves are used and once it reaches 5, the usage of these moves are no longer allowed. Essentially, it's a hard limit to this enemy where they cannot use either moves more than 5 times combined in the entire battle
- `data[1]`: The amount of times the reviving move was used. The value is incremented each time the reviving move is used. When the value reaches 3, the odds that the move is used when all other conditions are fufilled lowers to 20% from 70%. Essentially, it's a deterrent on the usage of the reviving move more than 3 times
- `data[2]`: A cooldown on the usage of the reviving move and the enemy party healing move. When either move is used, the value is set to 2. On every actor turns that either of these move isn't used, the value is decremented in the post move logic. For either of the moves to become usable again, the value must be 0 or below. Essentially, it's a 2 actor turn antispams where both moves can't be used again
- `data[3]`: A lock on the usage of the reviving move and the enemy party healing move. When the value is 1, it prevents either move to be used. The value is reset to 0 in the post move logic after any moves is used except these 2 moves. The lock is never activated by this enemy, but rather by [Delilah](Delilah.md) which sets it to 1 when they use their reviving move
- `data[4]`: This doesn't do anything meaningful in practice. It is set to 1 when the party wide sword jump attack is used and it is supposed to prevent the use of the charging move if the value is 1, but this doesn't work. This is because the value is always set to 0 on the same actor turn as the usage of the party wide sword jump attack move is used which undoes the effect it would have had.

## Move selection
8 moves are possible:

1. A single target downard sword slash that can inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md)
2. A single target triple downward sword slash
3. A single target sword hit shockwave attack
4. Prepares for a big attack by setting `charge` to 2
5. A single target sideway sword slash that can inflict [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md)
6. A party wide sword jump attack that can inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md)
7. Heals 15 `hp` and calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Delilah](Delilah.md)
8. Revives [Delilah](Delilah.md) leaving them at 7 `hp`

Move 7 and 8 are isolated in the move selection process and their usage consideration takes priority over anything else. 

### 7 and 8 selection process
To have a chance to use either move 7 or 8, all of the following conditions must be fufilled:

- `data[3]` is 0 (the lock on using either moves wasn't activated by [Delilah](Delilah.md))
- `charge` is 0 (this doesn't necessarily mean that move 4 was used previously as it is possible the player cleared this enemy's `charge`)
- `data[0]` is less than 5 (this enemy hasn't reached its limit of move 7 or 8's usage)
- `turns` is above 1 (at least 2 main turns passed since the start of the battle)
- `data[2]` is 0 or below (the cooldown for being able to use move 7 or 8 expired)

If all of the above are fufilled, it is now possible that move 7 or 8 is used. However, either requires specific conditions for the move to actually be used and it's still possible neither are. If neither are, the standard move selection process happens with move 1 through 6.

Move 8 is checked first and it will be used if all of the following conditions are fufilled:

- [Delilah](Delilah.md) isn't present in `enemydata` (they died)
- Either of the following is true:
    - `forceattack` is -1 (not taunted)
    - [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.3 and a 40% RNG check passes
- A 70% RNG check passes (20% instead if `data[1]` is at least 3 meaning this enemy already used this move 3 times before)

If move 8 was decided to not be used, move 7 is checked next. It's only used if all of the following conditions are fufilled:

- [Delilah](Delilah.md) is present in `enemydata`
- Both this enemy and [Delilah](Delilah.md) have an [HPPercent](../../Actors%20states/HPPercent.md) lower than 0.25
- A 35% RNG check passes

If neither moves are used, the standard move selection process is done instead.

### Standard move selection
Move 6 is always (and only used) if `basestate` is 106 (meaning move 4 was used before as it sets `basestate` to 106).

As for move 1 through 5, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/10|
|2|1/10|
|3|3/10|
|4|1/10|
|5|2/10|

However, move 4 has additional requirements for the move to actually be used once selected. The following conditions must all be fufilled for move 4 to be used once selected:

- [HPPercent](../../Actors%20states/HPPercent.md) is above 0.15 and below 0.75
- `cantmove` is 0 (meaning this is the last actor turn of the main turn for this enemy)
- `data[4]` is 0 (thid doesn't do anything in practice and it will always be true)

If the move selected, but the conditions above aren't fufilled, move 3 is used instead

## Pre move logic
The following logic is always done before the usage of a move:

- `walkstate` is set to 23 (`Chase`)
- If [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and `moves` is 1 (meaning this phase transition hasn't happened yet), this enemy goes into a phase transition where they get 2 actor turns per main turn including the current one:
    - `Charge18` sound plays with 1.1 pitch
    - y `spin` set to 10.0
    - animstate set to 4 `ItemGet`
    - Yield for 0.5 seconds
    - `spin` zeroed out
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 5 (yellow up arrow)
    - `StatUp` sound plays
    - animstate set to the local startstate
    - Yield for 0.5 seconds
    - `moves` set to 2 and `cantmove` is decremented which gives 2 actor turn per main turn to this enemy including the current one

## Post move logic
The following logic is always done after the usage of a move except fot the reviving move and the enemy party healing move:

- `data[4]` is set to 0 (this undoes any effect this value would have had, see the `data` section above for details)
- `data[2]` is decremented (the cooldown for the usage of the reviving move and the enemy party healing move)
- `data[3]` is set to 0 (resets the lock on the usage of the reviving move and the enemy party healing move that was activated by [Delilah](Delilah.md))

## Move 1 - Single downward sword slash
A single target downard sword slash that can inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|7|[DefDownOnBlock](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (3.0, 0.0, -0.15)
- Yield all frames until `forcemove` is done
- `Woosh6` sound plays
- animstate set to 102
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `WaspKingAxToss` sound plays with 0.95 pitch
- animstate set to 103
- `Thud4` sound plays with 1.1 pitch
- ShakeScreen called with 0.2 ammount and 0.75 time
- `impactsmoke` particles plays at `playertargetentity` position
- DoDamage 1 call happens
- Yield for 0.5 seconds
- Yield for 0.25 seconds

## Move 2 - Triple downward sword slash
A single target triple downward sword slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|
|2|Always happen 2 times after DoDamage 1|This enemy|The same target as DoDamage 1|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` + (3.0, 0.0, -0.15)
- Yield all frames until `forcemove` is done
- `Woosh6` sound plays
- animstate set to 102
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `WaspKingAxToss` sound plays with 0.95 pitch
- animstate set to 103
- `Thud4` sound plays with 1.1 pitch
- ShakeScreen called with 0.2 ammount and 0.75 time
- `impactsmoke` particles plays at `playertargetentity` position
- DoDamage 1 call happens
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (-0.45, 2.4, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- Done 2 times:
    - animstate set to 102
    - Yield for a frame
    - `WaspKingAxToss` sound plays with 0.95 pitch
    - animstate set to 103
    - ShakeScreen called with 0.2 ammount and 0.75 time
    - `impactsmoke` particles plays at `playertargetentity` position
    - DoDamage 1 call happens
    - `Thud4` sound plays with 1.1 pitch
    - Yield for 0.33 seconds
- Yield for 0.5 seconds

## Move 3 - Sword hit shockwave attack
A single target sword hit shockwave attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|6|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (8.5, 0.0, -0.1) with 1.5 multiplier
- Yield all frames until `forcemove` is done
- `flip` set to false
- `Woosh` sound plays
- animstate set to 102
- ShakeSprite called with 0.1 intensity and 45.0 frametimer
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 103
- `WaspKingAxToss` with 0.95 pitch
- `Thump5` sound plays
- ShakeScreen called with 0.1 ammount and -1.0 time (infinite)
- A new `Prefabs/Objects/StratosWave` GameObject is instantiated rooted
- Over the course of 21.0 frames, the `Prefabs/Objects/StratosWave` moves from this enemy position + (-2.0, 0.0, -0.1) to `playertargetentity` position - 0.1 in z via a lerp
- `HugeHit4` sound plays
- DeathSmoke particles plays at `playertargetentity` position with 4.0x size
- `impactsmoke` particles plays at `playertargetentity` position
- The `Prefabs/Objects/StratosWave` is moved offscreen at -9999.0 in y then destroyed in 1.0 second
- DoDamage 1 call happens
- Yield for 0.5 seconds
- MainManager.`screenshake` zeroed out

## Move 4 - Prepares a big attack with 2 `charge`
Prepares for a big attack by setting `charge` to 2. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `Charge18` sound plays with 1.1 pitch
- The local startstate, animstate and `basestate` set to 106 (this allows to use the party wide sword jump attack on the next actor turn when not using the reviving move or the enemy party healing move)
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- `StatUp` sound plays
- Yield for 0.5 seconds
- `charge` is set to 2

## Move 5 - Single sideway sword slash
A single target sideway sword slash that can inflict [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|7|[AtkDownOnBlock](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 104
- Yield for 0.5 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.0, 2.3, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- `trail` set to true
- Camera moves to look near `playertargetentity` position + (1.0, 0.5, 0.0)
- Over the course of 6.0 frames, this enemy moves to `playertargetentity` position + (2.5, 0.0, -0.1) via a lerp
- `trail` set to false
- `Slash2` sound plays
- animstate set to 105
- Yield for a frame
- `HugeHit` sound plays
- DoDamage 1 call happens
- `enemybounce` set to a new coroutine being a new BounceEnemy call on `playerdata[playertargetID]` with routineid 0 for 1 bounce with 1.2 multiplier
- Yield for 0.25 seconds
- Yield for 0.5 seconds
- Yield all frames until `enemybounce[0]` is null (the coroutine completed)

## Move 6 - Party wide sword jump attack
A party wide sword jump attack that can inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md).

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|5|[DefDownOnBlock](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- The local startstate is set to 13 (`BattleIdle`)
- `checkingdead` set to a new JumpPartyAttackCoroutine (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null
- `basestate` is set to the local startstate (this will prevent the usage of this move on the next actor turn)
- `data[4]` is set to 1 (this does nothing, see the `data` section above for details)

This is what the coroutine ends up doing:

- ShakeSprite called with intensity of (0.1, 0.0, 0.0) and 30.0 frametimer
- `Grow1` sound plays with 0.8 pitch
- Yield for 0.5 seconds
- Camera moves to look near `partymiddle`
- animstate set to 107
- `Jump` sound plays
- `Launch` sound plays
- Over the course of 51.0 frames, this enemy moves from its position with a y component of 0.0 to (-4.0, 0.0, 0.0) via a BeizierCurve3 with a ymax of 8.0
- animstate set to 103
- `Launch` sound plays
- `Explosion5` sound plays
- ShakeScreen called with an ammount of 0.2 and 0.5 time
- `impactsmoke` particles plays at this enemy position - 0.5 in x
- PartyDamage 1 call happens
- Yield for 0.75 seconds
- `checkingdead` set to null which signals the caller that this coroutine completed

## Move 7 - Heals 15 `hp` and [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Delilah](Delilah.md)
Heals 15 `hp` and calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Delilah](Delilah.md). No damages are dealt.

### Logic sequence

- `data[0]` is incremented (amount of times this this move and the enemy party healing one was used)
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `KingDinner` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by 15 `hp`
- [Heal](../../Actors%20states/Heal.md) called to heal [Delilah](Delilah.md) by 15 `hp`
- The `KingDinner` sprite gets destroyed
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on this enemy
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on `Delilah`
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `Delilah`'s `data[1]` is set to 1 which activates the lock on `Delilah`'s usage of any moves from their support items move selection set (see their [move selection](Delilah.md#move-selection) documentation for details)
- `data[2]` is set to 2 (the cooldown on the usage of this move and the enemy party healing one)

## Move 8 - Revives [Delilah](Delilah.md)
Revives [Delilah](Delilah.md) leaving them at 7 `hp`. No damages are dealt.

### Logic sequence

- `data[0]` is incremented (amount of times this this move and the enemy party healing one was used)
- `data[1]` is incremented (amount of times this move was used)
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `MagicDrops` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [ReviveEnemy](../ReviveEnemy.md) called with `reservedata` index 0 (`Delilah`) leaving them with 0.3% of `hp` with canmove
- Yield for a frame
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP counter) with an ammount of 7 starting from `enemydata[1]`'s position (should be `Delilah`) + 1.0 in y and ending to Vector3.up
- `enemydata[1]` (`Delilah`)'s `hp` is set to 7
- `Heal` sound plays
- `enemydata[1]` (`Delilah`) animstate set to 13 (`BattleIdle`)
- The `MagicDrops` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[2]` is set to 2 (the cooldown on the usage of this move and the enemy party healing one)
