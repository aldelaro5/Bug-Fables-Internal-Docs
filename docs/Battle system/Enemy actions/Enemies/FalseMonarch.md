# `FalseMonarch`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the party wide mothflies attack move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the crown throw move, the amount of frames the crown takes to reach its target changes to 19.0 frames from 26.0 frames
- In the crown throw move, the damageammount of the DoDamage call changes to 5 from 4
- In the multiple mothflies projectiles throw move, the amount of hits changes to be random from 3 to 5 inclusive instead of being random from 3 to 4 inclusive
- In the multiple mothflies projectiles throw move, there is now a 40% chance for the property of the Projectile call to be [Poison](../../Damage%20pipeline/AttackProperty.md)
- In the multiple mothflies projectiles throw move, the damage amount of the Projectile call changes to 3 from 2
- In the multiple mothflies projectiles throw move, the speed of the Projectile call changes to 20.0 from 27.0
- In the 2 [MothFly](Mothfly.md) summons move, after the summoning is done, every enemy party members other than this enemy now gets a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) and an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) conditions for 999999 main turns (effectively infinite)
- In the 2 [MothFly](Mothfly.md) summons move, after the summoning is done, this enemy's `cantmove` is decremented which gives them an additional actor turn
- In the charging move, this enemy now gets a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 2 main turns (effectively 1 main turn as the current main turn ends soon after)
- In the close range mothflies attack move, the move always inflicts the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition even if `commandsuccess` is false (didn't blocked, ignores FRAMEONE) as long as the target doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) or [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition
- In the close range mothflies attack move, if the move inflicts the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition, it is inflicted for 2 main turns (effectively 1 main turn since the current main turn ends soon after) if `commandsuccess` is true (blocked, ignores FRAMEONE). It is still inflicted for 3 main turns (effectively 2) if `commandsuccess` is false. The combined effect with the change above is blocking (ignoring FRAMEONE) now decrease the inflicting main turn count instead of preventing the inflicting entirely

## Move selection
6 moves are possible:

1. A single target crown throw that may inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md)
2. A multiple targets mothflies projectiles throw
3. Summons 2 new [Mothfly](Mothfly.md) enemies
4. Prepares for a party wide attack next actor turn by setting their `charge` to 2
5. A single target close range mothflies attack that may inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md)
6. A party wide mothflies attack

Move 6 is always (and only used) when `basestate` isn't 0 (`Idle`) which should means that move 4 was used on the previous actor turn as it sets the `basestate` away from 0.

For the other moves, the decision of which one to use is based on odds, but some moves have an requirement that must be fufilled after being selected. If the move is selected, but the requirement isn't fufilled, a continue directive is issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection process. Here are the odds and requirements:

|Move|Odds|Requirement|
|---:|----|----------|
|1|2/12|None, the move is always used when selected|
|2|3/12|None, the move is always used when selected|
|3|3/12|This enemy must be the last `enemydata` remaining|
|4|2/12|[HPPercent](../../Actors%20states/HPPercent.md) must be 0.7 or lower|
|5|2/12|None, the move is always used when selected|

## Move 1 - Crown throw
A single target crown throw that may inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4 (5 instead if hardmode is true)|null|null|`commandsuccess`|

### Logic sequence

- `Toss16` sound plays
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]` + 3.5
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (6.5, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 105
- Yield for a random interval between 0.5 and 0.75 seconds
- `Toss15` sound plays
- animstate set to 107
- A new sprite object is created rooted using the `projectilepsrites[14]` sprite (a yellow crown) at this enemy position + (-0.5, 2.0. -0.1) with a SpinAround that has an `itself` of (0.0, 0.0, -30.0)
- If `hologram` is true, the material of the crown's SpriteRenderer is set to `holosprite`
- Over the course of 26.0 frames (19.0 frames instead if hardmode is true), the crown moves to `playertargetentity` position + (0.0, 1.25, -0.1) via a lerp
- `MetalHit` sound plays
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE) and `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) or [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition:
    - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition on `playerdata[playertargetID]` for 3 main turns (effectively 2 main turns since the current main turn advances soon) with effect
- animstate set to 109
- Over the course of 31.0 frames, the crown moves to this enemy position + (0.0, 2.0, -0.1) via a BeizierCurve3 with a ymax of 4.0
- animstate set to 110
- The crown object gets destroyed
- Yield for 0.25 seconds
- The local startstate is set to `basestate`

## Move 2 - Mothflies projectiles throw
A multiple targets mothflies projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 3 to 4 times inclusive determined randomly (3 to 5 times inclusive instead if hardmode is true), but each call can only happens if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable returns doesn't return -1|2 (3 instead if hardmode is true)|null (if hardmode is true, this has 40% chance to be [Poison](../../Damage%20pipeline/AttackProperty.md) instead)|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup> (target changes for each calls)|A new `Prefabs/Objects/MothflyLite` GameObject rooted positioned at this enemy + (0.0, 1.0, -0.1) whose SpriteRenderer uses the `holosprite` material if `hologram` is true and with a pure magenta color if the porperty is [Poison](../../Damage%20pipeline/AttackProperty.md) with a ShadowLite that is SetUp with 0.5 opacity and 0.1 size|27.0 (20.0 instead if hardmode is true)|0.0|`keepcolor,wait@0.5,anim@Attack,destoffscr@30@5`|null|null|`@Toss14`|Vector3.zero|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- `Toss15` sound plays
- animstate set to 100
- Yield for 0.5 seconds
- `Toss16` sound plays
- animstate set to 103
- Yield for 0.25 seconds
- The amount of hits is determined which is random between 3 and 4 times (between 3 and 5 times instead if hardmode is true). A new Transform array is initialised with this amount as the length to hold the projectiles objects
- For each hit to do:
    - [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getrandomavaliableplayer) with nullable is called and if it doesn't return -1:
        - `Toss13 `sound plays
        - The projectile element is set to a new `Prefabs/Objects/MothflyLite` GameObject rooted positioned at this enemy + (0.0, 1.0, -0.1) whose SpriteRenderer uses the `holosprite` material if `hologram` is true and with a pure magenta color if the porperty is [Poison](../../Damage%20pipeline/AttackProperty.md) with a ShadowLite that is SetUp with 0.5 opacity and 0.1 size
        - MainManager.MoveTowards is called to move the projectile to Random.insideUnitCircle * 2.0 + 4.0 in y over 20.0 frames with smooth without local (this is done in paralel)
        - Projectile 1 call happens
        - Yield for a random interval between 0.45 and 0.75 seconds
- Yield all frames until all projectiles elements are null (all Projectile 1 calls completed)
- The local startstate is set to `basestate`

## Move 3 - Summons 2 [Mothfly](Mothfly.md)
Summons 2 new [Mothfly](Mothfly.md) enemies. No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affects anything for this move.

### Logic sequence

- `Toss10` sound plays
- `checkingdead` set to a new SummonMothFly coroutine (`checkingdead` is set to null when it completes)
- Yield all frames until `checkingdead` is null
- If hardmode is true:
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called twice on every enemy party members except this enemy to inflict them the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) and [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) conditions for 999999 main turns each (infinite)
    - `cantmove` is decremented on this enemy which grants tham an additional actor turn
- The local startstate is set to `basestate`

This is what the SummonMothfly coroutine ends up doing:

- Camera moves to look near this enemy, but zoomed in
- animstate set to 101
- Yield for 0.65 seconds
- `rotater` scale increased by 30%
- Done 2 times:
    - [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to a new [Mothfly](Mothfly.md) enemy at this enemy + (0.0, `height`, 0.1)
    - Yield a frame
    - The `enemydata[lastaddedid]` gets some adjustements:
        - `maxhp` and `hp`: divided by 2 floored
        - `exp`: 0
        - `money`: 0
        - y `spin`: 20.0
- Over the course of 31.0 frames, `rotater` scale changes to its original value before this coroutine via a lerp and the 2 new enemies that were just summoned moves from this enemy position to (`x`, 0.0, -0.1) via a BeizierCurve3 with a ymax of 4.0 where `x` is this enemy x position - 2.5 for the first enemy and this enemy x position + 2.5 for the second enemy
- `rotater` scale is set to its value before this coroutine
- Both enemies who were just summoned has their `spin` zeroed out
- `checkingdead` is set to null which signals the caller that this coroutine completed

## Move 4 - Prepares attack with 2 `charge`
Prepares for a party wide attack next actor turn by setting their `charge` to 2. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affects anything for this move.

### Logic sequence

- `Charge19` sound plays
- animstate set to 111
- Camera moves to look near this enemy
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- If hardmode is true, [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy for 2 main turns (effectively 1 main turn due to the current main turn advancing soon)
- `StatUp` sound plays
- `charge` set to 2
- Yield for 0.5 seconds
- `basestate` set to 111 which allows this enemy to use the party wide mothflies attack move on their next actor turn
- The local startstate is set to `basestate`

## Move 5 - Close range mothflies attack
A single target close range mothflies attack that may inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4 (5 instead if hardmode is true)|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Toss15` sound plays
- animstate set to 100
- Yield for a random interval between 0.25 and 0.5 seconds
- animstate set to 101
- `MothflyATK` sound plays
- Yield for 0.25 seconds
- `MothflyAttack` particles plays at this enemy position + (-0.25, 1.0, -0.1) with 5.0 alivetime
- DoDamage 1 call happens
- If hardmode is true or `commandsuccess` is false (didn't blocked, ignores FRAMEONE) and `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) or [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition:
    - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition on `playerdata[playertargetID]` for 3 main turns (2 main turns instead if hardmode and `commandsuccess` are true) with effect. This is effectively 1 main turns less because the current main turn will be advanced soon
- Yield for 0.5 seconds
- The local startstate is set to `basestate`

## Move 6 - Party wide mothflies attack
A party wide mothflies attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|5 (3 instead if `barfill` is higher than 0.95 after DoCommand 1 meaning the bar got almost filled up)|null|true if `barfill` is higher than 0.95 after DoCommand 1 meaning the bar got almost filled up, false otherwise|0.0|Vector3.zero|false|null|

### Logic sequence

- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (0.5, 0.0, 0.0) with 2.0 multiplier
- Camera moves to look near the midpoint between (0.5, 0.0, 0.0) and `partymiddle`
- Yield all frames until `forcemove` is done
- `Toss15` sound plays
- animstate set to 100
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 101
- `MothflyLong` sound plays
- Yield for 0.25 seconds
- A new `Prefabs/Particles/MothflyAttack` GameObject is instantiated rooted at this enemy position + (-0.25, 1.0, -0.1) with its ParticleSystem's MainModule getting some adjustements:
    - loop: true
    - simulationSpace: World
    - startLifetime: 2.5
    - startSpeed: A constant curve with value 7.0
    - gravityModifier: A constant curve with value 0.0
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the [TappingKey](../../Action%20commands/TappingKey.md) command's help)
- DoCommand 1 call happens
- All player party member who isn't [stopped](../../Actors%20states/IsStopped.md) has their y `spin` set to 15.0
- Yield all frames until `doingaction` goes to false (DoCommand 1 is done). Before each frame yield, all player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`) (but if `blockcooldown` is above 0 meaning a block was done, it's set to 24 (`Block`) instead)
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- All player party member has their `spin` getting zeroed out
- PartyDamage 1 call happens
- The `Prefabs/Particles/MothflyAttack` object gets moved offscreen at -9999.0 in y then destroyed in 5.0 seconds
- Yield for 0.5 seconds
- `basestate` set to 0 (`Idle`) so this move isn't used on the next actor turn
- The local startstate is set to `basestate`
