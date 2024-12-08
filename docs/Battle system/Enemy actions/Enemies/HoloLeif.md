# `HoloLeif`

## Assumptions
This enemy is assumed to be fought with [HoloVi](HoloVi.md) and [HoloKabbu](HoloKabbu.md) for certain moves to be usable and for the move selection process to work correctly.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 5 element with a starting value of 0.

- `data[0]`: This reperesents the amount of time some specific moves were used in the battle by incrementing the value on each usage. If the value reaches 5, this enemy will no longer be able to use any moves in the list. Here are the implicated moves:
    - Revives another enemy party member from `reservedata`
    - Heal another enemy party member for 7 `hp`
    - A single target attack involving the same effects as the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) action where the target is a player party member
- `data[1]`: An actor turn cooldown on the usage of the any moves managed by `data[0]`. On usage of any of these moves, the value is set to 3. The value is always decremented in the pre move logic when above 0 and the value must be 0 for any of the 3 moves managed by `data[0]` to be usable. Effectively, it's a 2 actor turns cooldown before any of the 3 moves become usable again
- `data[2]`: UNUSED. This is because that's because this enemy doesn't have a turn relay simulation move unlike [HoloVi](HoloVi.md) and [HoloKabbu](HoloKabbu.md). Since these 2 enemnies still needs to set `data[2]` on the receiver of the relay, this slot is allocated, but not used in any way. It is only set when either of these 2 enemies uses their turn relay simulation move to 2 and it is decremented at the start of the actor turn if above 0, but it is not used to influence move selection
- `data[3]`: An actor turn cooldown on the usage of some specific moves. The value is always decremented in the pre move logic when above 0 and the value must be 0 for any of moves to be usable. It is set to 2 every time any of the moves is used. Effectively, it's a 1 actor turn cooldown before any of the moves become usable again. Here are the affected moves:
    - Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to all enemy party members including themselves
    - Gives the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on an enemy party member
    - Gives the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition to all alive player party members
    - Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to all enemy party members
    - Gives the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition to all alive player party members
- `data[4]`: UNUSED. The only thing that happens with this value is it is decremented at the start of the actor turn if above 0, but it is never set to a value above it and it is not used in any way

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
The following all happens in the HoloLeif coroutine:

- The first `enemydata` with an [animid](../../../Enums%20and%20IDs/AnimIDs.md) of `Beetle` is obtained (should be [HoloKabbu](HoloKabbu.md) under normal gameplay) and if it exists, [ItemSpinAnim](../../Visual%20rendering/ItemSpinAnim.md) is called on them with a pos of their position + 1.0 in y using the `FavoriteOne` [medal](../../../Enums%20and%20IDs/Medal.md) icon as sprite and without playSound
- Yield return on a EnemyAngryCharge call with this enemy's battleentity:
    - Set `dontusecharge` to true
    - If `charge` is less than 3, it is incremented with the following:
        - animstate set to 102
        - `Wam` sound plays
        - [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote with 20 time
        - Yield all frames until `emoticoncooldown` expires
        - `StatUp` sound plays
        - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
        - Yield for 0.5 seconds
        - `charge` is incremented

## Move selection
12 moves are possible:

1. A single target ice attack
2. Revives another enemy party member from `reservedata` leaving them at 7 `hp` with canmove (so they can take an actor turn immediately on the same main turn)
3. Heal an enemy party member (including themselves) for 7 `hp`
4. A single target attack involving the same effects as the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) action where the target is a player party member
5. A party wide icicle throw
6. Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to all enemy party members including themselves for 3 main turns (effectively 2 main turns since the current main turn ends soon)
7. A multiple hits icicle fall attack that hits 4 times where each hit is single target
8. Calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on a player party member
9. Gives the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on an enemy party member (including themselves)
10. Gives the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition to all alive player party members for 3 main turns (effectively 2 main turns since the current main turn ends soon)
11. Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to all enemy party members including themselves for 3 main turns (effectively 2 main turns since the current main turn ends soon)
12. Gives the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition to all alive player party members for 3 main turns (effectively 2 main turns since the current main turn ends soon)

If `turns` is 0 (the first main turn), the selected move is alwauys deterministic and is determined with the following logic:

- If this enemy's `cantmove` is -1, move 6 is used. This means this enemy has 2 actor turns available which can only happen if [HoloVi](HoloVi.md) or [HoloKabbu](HoloKabbu.md) used their turn relay simulation move on this enemy on the first main turn
- Otherwise, move 5 is used

Otherwise (not on the first main turn), the decision of which move to use is based on weighted odds. However, these weights have 2 possible distribution sets which will be numbered as follows:

1. [HoloVi](HoloVi.md) is present (assumed to not be dead yet)
2. [HoloVi](HoloVi.md) is not present (assumed to be dead)

Also, each moves have additional requirements that if they aren't fufilled, the move won't be used and instead cause a reroll of the move selection process. However the amount of attempts to a move selection is tracked and after 5 failures to select a move, move 1 is always used.

Here are the odds for all distributions sets, their usage requirements and the step to take if those requirements aren't fufilled.

|Move|Odds set 1|Odds set 2|Requirements|
|---:|----------|----------|------------|
|1|0/32|8/60|None, the move is always used when selected|
|2|2/32|2/60|<ul><li>`data[0]` is less than 5 (move 2, 3 and 4 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2, 3 and 4 weren't used on the last 2 actor turns of this enemy)</li><li>`reservedata` isn't empty (there is someone available to be revived)</li></ul>|
|3|1/32|1/60|<ul><li>`data[0]` is less than 5 (move 2, 3 and 4 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2, 3 and 4 weren't used on the last 2 actor turns of this enemy)</li><li>There is at least 1 enemy party member (including themselves) whose `hp` is above 0, but their [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.8</li></ul>|
|4|1/32|1/60|None, the move is always used when selected|
|5|2/32|8/60|None, the move is always used when selected|
|6|4/32|4/60|`data[3]` is 0 or less (move 6, 9, 10, 11 and 12 weren't used on the last actor turn of this enemy)|
|7|2/32|20/60|None, the move is always used when selected|
|8|4/32|4/60|There is at least 1 player party member whose `hp` is above 0 and either they have more than 0 `charge` or they have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|
|9|4/32|4/60|`data[3]` is 0 or less (move 6, 9, 10, 11 and 12 weren't used on the last actor turn of this enemy)|
|10|4/32|4/60|`data[3]` is 0 or less (move 6, 9, 10, 11 and 12 weren't used on the last actor turn of this enemy)|
|11|4/32|4/60|`data[3]` is 0 or less (move 6, 9, 10, 11 and 12 weren't used on the last actor turn of this enemy)|
|12|4/32|0/60|`data[3]` is 0 or less (move 6, 9, 10, 11 and 12 weren't used on the last actor turn of this enemy)|

As a sidenote, the entire move selection process occurs in the HoloLeif coroutine who is yield returned by DoAction.

## Pre move logic
The following logic always happens before the usage of a move (this happens in DoAction, the rest happens in HoloLeif):

- entity.`sprite`.material.renderQueue set to 2500
- CreateShield called on the entity which initialises their `bubbleshield` if it didn't exist yet

## Move 1 - Single target ice attack
A single target ice attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Freeze](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyIceAttackBasic coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)
- animstate set to 108
- `IceMothThrow` sound plays
- A new `Prefabs/Objects/SingleSphere` GameObject is created childed to a new `Prefabs/AnimSpecific/mothbattlesphere` GameObject
- Over the course of 31.0 frames, the `Prefabs/AnimSpecific/mothbattlesphere` moves from this enemy position + (1.1, 1.85, -0.1) to this enemy position + (-2.0, -0.5, 5.0) via a BeizierCurve3 with a ymax of 5.0
- The `Prefabs/Objects/SingleSphere` gets rooted
- If `hologram` is true, the `Prefabs/Objects/SingleSphere` Renderer's color is set to 8080FF (mostly blue) with half opacity
- `Prefabs/AnimSpecific/mothbattlesphere` gets destroyed
- `WatcherIce` sound plays
- Over the course of 81.0 frames (71.0 instead if hardmode is true), `Prefabs/Objects/SingleSphere` moves from its position with the y component at 0.0 to `playertargetentity` position via a SmoothLerp
- `Prefabs/Objects/SingleSphere` gets destroyed
- animstate set to 111
- `IceMothHit` sound plays
- A new `Prefabs/Particles/mothicenormal` GameObject is created rooted at `playertargetentity` position + 0.5 in y then destroyed in 2.0 seconds
- A new `Prefabs/Objects/icepillar` is created rooted at `playertargetentity` position with a DialogueAnim that has a `targetscale` of (0.5, 0.5, 1.0)
- DoDamage 1 call happens
- Yield for 0.65 seconds
- The `Prefabs/Objects/icepillar`'s DialogueAnim has its `shrink` set to true with a `shrinkspeed` of 0.025
- The `Prefabs/Objects/icepillar` gets destroyed in 5.0 seconds
- Yield for 0.5 seconds

## Move 2 - Revives another enemy party member leaving them at 7 `hp`
Revives another enemy party member from `reservedata` leaving them at 7 `hp` with canmove (so they can take an actor turn immediately on the same main turn). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Yield return ItemAnim with the battleentity with the `MagicDrops` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- `data[0]` is incremented. If it reaches 5, this move, move 2 and move 4 can't be used anymore
- `data[1]` is set to 3 so this move, move 2 and move 4 can't be used for the next 2 actor turns
- [ReviveEnemy](../ReviveEnemy.md) called to revive a random `reservedata` with an hppercent of 7.0 (leaves them at 7 `hp`) with canmove and showcounter
- Yield for 1.0 second

## Move 3 - Heal another enemy party member for 7 `hp`
Heal an enemy party member (including themselves) for 7 `hp`. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Yield return ItemAnim with the battleentity with the `JaydeStew` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- `data[0]` is incremented. If it reaches 5, this move, move 2 and move 4 can't be used anymore
- `data[1]` is set to 3 so this move, move 2 and move 4 can't be used for the next 2 actor turns
- [Heal](../../Actors%20states/Heal.md) called on a random enemy party member to heal them for 7 `hp`. The selection of the enemy party member to heal only includes those whose `hp` is above 0, but their [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.8 (this includes this enemy)
- Yield for 1.0 second

## Move 4 - [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action
A single target attack involving the same effects as the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) action where the target is a player party member.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|null|`playerdata[playertargetID]` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|6|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|empty array|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- Yield return ItemAnim with the battleentity with the `LonglegSummoner` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Yield return a LongLeg coroutine with this enemy's battleentity and `playerdata[playertargetID]` as the target:
    - Camera moves slowly towards the target slightly to the left, up and behind
    - animstate set to 28 (`TossItem`)
    - A new sprite is created rooted using the `LonglegSummoner` [item](../../../Enums%20and%20IDs/Items.md)'s sprite positioned at this enemy + (0.25, 2.25, -0.1)
    - `Toss` sound plays
    - Over the course of 51.0 frames, the sprite moves to the target position + (1.5, 0.0, 1.25) via a BeizierCurve3 with a ymax of 5.0. Before each frame yield, the sprite's z angle is increased by 10.0 * MainManager.`framestep`
    - animstate reset to the value before this action
    - `LongLegsGrow` sound plays
    - `Prefabs/Objects/LongLegs` instantiated offscreen with 0.0x scale
    - DeathSmoke particles plays at the sprite position
    - The sprite is destroyed
    - Yield for a frame
    - `Prefabs/Objects/LongLegs` position set to where the sprite was before its destruction
    - Over the course of 31.0 frames, the `Prefabs/Objects/LongLegs` moves to + 4.75 in y via a SmoothLerp and its scale changes to (1.0, 1.0, -1.0) via a SmoothLerp
    - The `Stomp` animation clip plays from `Prefabs/Objects/LongLegs`
    - `Grow1` sound plays
    - Yield for 0.75 seconds
    - The `Stomp2` animation clip plays from `Prefabs/Objects/LongLegs`
    - `longLegsStomp` sound plays
    - Yield for 0.15 seconds
    - DoDamage 1 call happens
    - Yield for 0.75 seconds
    - Over the course of 31.0 frames, the `Prefabs/Objects/LongLegs` moves to - 4.75 in y via a SmoothLerp and its scale changes to 0.0x via a SmoothLerp
    - DeathSmoke particles plays at the `Prefabs/Objects/LongLegs` position
    - `ChargeDown2` sound plays
    - `Prefabs/Objects/LongLegs` is destroyed
    - Yield for 0.5 seconds
    - `checkingdead` is set to null which signals the caller that the coroutine is done
- `data[0]` is incremented. If it reaches 5, this move, move 2 and move 4 can't be used anymore
- `data[1]` is set to 3 so this move, move 2 and move 4 can't be used for the next 2 actor turns

## Move 5 - Party wide icicle throw
A party wide icicle throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Freeze](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence
This is done by yield returning the EnemyIceFall coroutine with the battleentity:

- animstate set to 105
- A new `Prefabs/Objects/icecle` GameObject is created rooted positioned at this enemy position + (-2.0, 4.5, 0.0) with a scale of 0.0x and its BoxCollider destroyed
- `Spin` sound plays on loop using `sounds[8]`
- Over the course of 60.0 frames, `Prefabs/Objects/icecle` scale changes to 1.25x via a lerp, the y angle goes towards 10.0 and the `sounds[8]` pitch changes from 0.0 to 1.0
- A new UI object is created named `t` childed to the `GUICamera` using the `guisprites[41]` sprite (a crosshair) on layer 15 (`3DUI`)
- animstate set to 107
- `Crosshair` sound plays on loop using `sounds[9]` with 0.9 pitch and 0.35 volume
- Over the course of 81.0 frames, `t` moves from (0.0, 3.0, 0.0) to (-4.5, 1.0, 0.0) via a SmoothLerp + a number in the y component that goes from Sin(Time.time * 2.0) * -3.0 to 0.0. Before each yield, `t` z angle increases by 5x the game's frametime and `Prefabs/Objects/icecle` y angles increases by 10.0
- `sounds[8]` and `sounds[9]` stopped
- `IceMothThrow` sound plays
- `t` gets destroyed
- `Prefabs/Objects/icecle` angles set to (0.0, 0.0, -45.0)
- Over the course of 26.0 frames, `Prefabs/Objects/icecle` moves to (-4.5, 1.0, 0.0) via a lerp
- `IceMothHit` sound plays
- `mothicenormal` particles plays at the `Prefabs/Objects/icecle` position with a scale of 2.0x
- `Prefabs/Objects/icecle` gets destroyed
- PartyDamage 1 call happens
- Yield for 0.5 seconds

## Move 6 - Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to all enemy party members
Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to all enemy party members including themselves for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyLeifBuffDebuff coroutine with the battleentity and the type being the `EmpowerPlus` [Skill](../../../Enums%20and%20IDs/Skills.md):

- `dontusecharge` is set to true
- `Magic` sound plays
- `Prefabs/Particles/MagicUp` instantiated rooted at this enemy + 0.5 in y
- animstate set to 4 (`ItemGet`)
- y `spin` set to -20.0 (this is ignored soon)
- All enemy party members (including this enemy) has their y `spin` set to -15.0
- Yield for 0.75 seconds
- All enemy party members (including this enemy) has their `spin` zeroed out followed by [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to give them the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 3 main turns with effect (effectively 2 main turns since the current main turn ends soon)
- `Prefabs/Particles/MagicUp` moved offscreen at 999.0 in y
- `spin` zeroed out
- `Prefabs/Particles/MagicUp` destroyed in 5.0 seconds

After the yield return, `data[3]` is set to 2 so this move alongside move 9, 10, 11 and 12 can't be used on the next actor turn.

## Move 7 - Multiple hits icicle fall attack
A multiple hits icicle fall attack that hits 4 times where each hit is single target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 4 times, but each call can only happen if at least 1 player party member is alive (`hp` more than 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|5|[Freeze](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyIceRain coroutine with the battleentity:

- Some camera fields changes to look closer to the player party:
    - `camoffset`: increases by (-1.5, 1.25, 1.0)
    - `camtargetpos`: Vector3.zero
- animstate set to 105
- Yield for 0.5 seconds
- The following happens up to 4 times (the iteration doesn't happen if all player party member have an `hp` of 0 or are [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `Prefabs/Objects/icecle` instantiated rooted at (`playertargetentity` x position, 15.0, 0.0)
    - `Toss4` sound plays
    - animstate set to 100
    - Over the course of 50.0 frames, `Prefabs/Objects/icecle` position is lerped to `playertargetentity` position + 1.0 in y
    - DoDamage 1 call happens
    - `mothicenormal` particles plays at `playertargetentity` position + 1.0 in y with a scale of 2.0x
    - `IceMothHit` sound plays
    - `Prefabs/Objects/icecle` gets destroyed
    - Yield for 0.33 seconds
    - animstate set to 105
- animstate set to 102
- Yield for 0.5 seconds

## Move 8 - Calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on a player party member
Calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on a player party member. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Select a random player party members as the target that fufill all of the following conditiosn (as explained in the move selection above, at least one target exists for this move to be used):
    - Their `hp` is above 0
    - They either have more than 0 `charge` or they have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- This enemy animstate set to 111
- `FastWoosh` sound plays
- Yield for 0.5 seconds
- `Heal3` sound plays
- The target animstate set to 11 (`Hurt`)
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on the target
- DeathSmoke particles plays at the target position with a size of 3.0x
- Yield for 1.0 second

## Move 9 - Gives the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on another enemy party member
Gives the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on an enemy party member (including themselves). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `data[3]` is set to 2 so this move alongside move 6, 10, 11 and 12 can't be used on the next actor turn
- Some camera fields changes to look closer at this enemy:
    - `camtargetpos`: This enemy position + 2.0 in x
    - `camspeed`: 0.01
    - `camoffset`: (0.0, 2.65, -7.0)
- animstate set to 102
- Yield for 0.75 seconds
- animstate set to 119
- Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Shield` sound played
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to give the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition to a random enemy party member whose `hp` is above 0 (including themselves) for 2 main turns (effectively 1 main turn since the current main turn ends soon)
- Yield for 0.75 seconds

## Move 10 - Gives the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition to all alive player party members
Gives the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition to all alive player party members for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyLeifBuffDebuff coroutine with the battleentity and the type being the `AttackDownPlus` [Skill](../../../Enums%20and%20IDs/Skills.md):

- `dontusecharge` is set to true
- `Magic` sound plays
- `Prefabs/Particles/MagicUp` instantiated rooted at this enemy + 0.5 in y
- animstate set to 4 (`ItemGet`)
- y `spin` set to -20.0 (this is ignored soon)
- All alive player party members (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) has their y `spin` set to -15.0
- Yield for 0.75 seconds
- All alive player party members (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) has their `spin` zeroed out followed by [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to give them the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition for 3 main turns with effect (effectively 2 main turns since the current main turn ends soon)
- `Prefabs/Particles/MagicUp` moved offscreen at 999.0 in y
- `spin` zeroed out
- `Prefabs/Particles/MagicUp` destroyed in 5.0 seconds

After the yield return, `data[3]` is set to 2 so this move alongside move 6, 9, 11 and 12 can't be used on the next actor turn.

## Move 11 - Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to all enemy party members
Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to all enemy party members including themselves for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyLeifBuffDebuff coroutine with the battleentity and the type being the `DefenseUpPlus` [Skill](../../../Enums%20and%20IDs/Skills.md):

- `dontusecharge` is set to true
- `Magic` sound plays
- `Prefabs/Particles/MagicUp` instantiated rooted at this enemy + 0.5 in y
- animstate set to 4 (`ItemGet`)
- y `spin` set to -20.0 (this is ignored soon)
- All enemy party members (including this enemy) has their y `spin` set to -15.0
- Yield for 0.75 seconds
- All enemy party members (including this enemy) has their `spin` zeroed out followed by [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to give them the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 3 main turns with effect (effectively 2 main turns since the current main turn ends soon)
- `Prefabs/Particles/MagicUp` moved offscreen at 999.0 in y
- `spin` zeroed out
- `Prefabs/Particles/MagicUp` destroyed in 5.0 seconds

After the yield return, `data[3]` is set to 2 so this move alongside move 6, 9, 10 and 12 can't be used on the next actor turn.

## Move 12 -  Gives the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition to all alive player party members
Gives the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition to all alive player party members for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyLeifBuffDebuff coroutine with the battleentity and the type being the `DefenseBreakAll` [Skill](../../../Enums%20and%20IDs/Skills.md):

- `dontusecharge` is set to true
- `Magic` sound plays
- `Prefabs/Particles/MagicUp` instantiated rooted at this enemy + 0.5 in y
- animstate set to 4 (`ItemGet`)
- y `spin` set to -20.0 (this is ignored soon)
- All alive player party members (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) has their y `spin` set to -15.0
- Yield for 0.75 seconds
- All alive player party members (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) has their `spin` zeroed out followed by [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to give them the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition for 3 main turns with effect (effectively 2 main turns since the current main turn ends soon)
- `Prefabs/Particles/MagicUp` moved offscreen at 999.0 in y
- `spin` zeroed out
- `Prefabs/Particles/MagicUp` destroyed in 5.0 seconds

After the yield return, `data[3]` is set to 2 so this move alongside move 6, 9, 10 and 11 can't be used on the next actor turn.
