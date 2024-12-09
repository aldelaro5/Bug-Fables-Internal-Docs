# `HoloKabbu`

## Assumptions
This enemy is assumed to be fought with [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) for certain moves to be usable and for the move selection process to work correctly.

It is also assume that this enemy's [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) contains [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) because of specific logic that involves their `hitaction` logic expecting it to be triggered by hitting this enemy (more specifically, an enemy with an [animid](../../../Enums%20and%20IDs/AnimIDs.md) of `Beetle`).

By the same token, it is assumed that this enemy's `animid` is `Beetle`.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 5 element with a starting value of 0.

- `data[0]`: This reperesents the amount of time some specific moves were used in the battle by incrementing the value on each usage. If the value reaches 5, this enemy will no longer be able to use any moves in the list. Here are the implicated moves:
    - Revives [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) from `reservedata`
    - Revives another enemy party member from `reservedata` leaving them at 7 `hp`
    - Inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition on [HoloVi](HoloVi.md) bypassing the `poisonres` check for 3 main turns. NOTE: For this move specifically, `data[0]` will NOT increment, but the move still won't be used if `data[0]` reaches 5
- `data[1]`: An actor turn cooldown on the usage of the any moves managed by `data[0]`. On usage of 2 of the 3 moves, the value is set to 3. The value is always decremented in the pre move logic when above 0 and the value must be 0 for any of the 3 moves managed by `data[0]` to be usable. Effectively, it's a 2 actor turns cooldown before any of the 3 moves become usable again. NOTE: The [Poison](../../Actors%20states/BattleCondition/Poison.md) infliction on [HoloVi](HoloVi.md) move exceptionally does NOT change the value of `data[1]` upon usage, but the usage of the move is still forbidden if `data[1]` is above 0
- `data[2]`: An actor turn cooldown on the usage of the turn relay simulation move. On every usage of the move, the value is set to 3 and it is always decremented in the pre move logic when above 0. For the move to be usable, the value needs to be 0. Effectively, it's a 2 actor turn cooldown on the move before it becomes usable again. NOTE: If [HoloVi](HoloVi.md) uses their turn relay simulation move to this enemy, this enemy's `data[2]` is set to 2 which is a 1 actor turn cooldown on the usage of this enemy's turn relay simulation move
- `data[3]`: This enforces a 1 time usage constraint of the move that revives [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) from `reservedata`. On usage of the move, the value is set to 1 while the move requires a value of 0. This means that once the move is used once in the battle, it is not usable again for the remainder of the battle. In these circumstances, the singular 7 `hp` revive move will be used instead when applicable
- `data[4]`: An actor turn cooldown on the usage of the move with a [Taunted](../../Actors%20states/BattleCondition/Taunted.md) infliction on all player party members for 2 main turns. When the move is used, the value is set to 3 and it is always decremented in the pre move logic if it is above 0. For the move to be usable, the value needs to be 0. Effectively, this is a 2 actor turns cooldown before the move becomes usable again

## Move selection
10 moves are possible:

1. A single target horn slash
2. Revives [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) from `reservedata` leaving their at 6 `hp` with canmove (so they can take an actor turn immediately on the same main turn)
3. Revives another enemy party member from `reservedata` leaving them at 7 `hp` with canmove (so they can take an actor turn immediately on the same main turn)
4. Inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition on [HoloVi](HoloVi.md) bypassing the `poisonres` check for 3 main turns (effectively 2 main turns since the current main turn ends soon)
5. Inflicts the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on all player party members for 2 main turns (effectively 1 main turn since the current main turn ends soon)
6. Revives another enemy party member from `reservedata` leaving them at 4 `hp` with canmove (so they can take an actor turn immediately on the same main turn)
7. A party wide dash attack
8. A single target rock throw
9. Simulate a turn relay by decrementing the `cantmove` of [HoloVi](HoloVi.md) or [HoloLeif](HoloLeif.md) which grants them an additional actor turn on this main turn
10. A single target underground strike

Move 5 is always used if `turns` is 0 (the first main turn).

Otherwise, the decision of which move to use is based on the weigthed odds. However, these weighted odds have 3 possible distribution sets which will be numbered as follows (the first one that applies is used):

1. All of the following conditions are fufilled:
    - Both [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) are present
    - This enemy has no `charge`
    - This enemy doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
2. Both [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) are present, but this enemy either has at least 1 `charge` or it has the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
3. Either [HoloVi](HoloVi.md) or [HoloLeif](HoloLeif.md) isn't present (assumed to be dead)

In all sets, move 2 and 3 belong together as one unit used in the move selection process. To simplify this fact, it will be refered to as move 2-3 when refering to the move selection.

Also, each moves have additional requirements that if they aren't fufilled, the move won't be used and instead cause a reroll of the move selection process. However the amount of attempts to a move selection is tracked and after 5 failures to select a move, move 1 is always used.

There is a special case for move 2-3 because when selected, the decision if either move can be used and which one to use is nested inside the main move selection. This means that when move 2-3 is selected, the following decision logic happens to handle this:

- If both [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) aren't present (assumed to be dead) and `data[3]` is 0 (move 2 wasn't used before), move 2 is always used. Due to `data[3]`, this can only happen once per battle
- If the above conditions aren't fufilled, but the `reservedata` isn't empty still (meaning it's assumed either [HoloVi](HoloVi.md) or [HoloLeif](HoloLeif.md) is dead), move 3 is used
- If none of the conditions above are fufilled, the move selection process is rerolled

Here are the odds for all distributions sets, their usage requirements and the step to take if those requirements aren't fufilled.

|Move|Odds set 1|Odds set 2|Odds set 3|Requirements|
|---:|----------|----------|----------|------------|
|1|6/42|6/39|0/11|None, the move is always used when selected||
|2-3|4/42|2/39|2/11|<ul><li>`data[0]` is less than 5 (move 2 and 3 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2 and 3 weren't used on the last 2 actor turns of this enemy)</li></ul>Also, the decision logic mentioned above applies which has additional requirements|
|4|2/42|1/39|1/11|<ul><li>`data[0]` is less than 5 (move 2 and 3 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2 and 3 weren't used on the last 2 actor turns of this enemy)</li><li>[HoloVi](HoloVi.md) is present and they do not have the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition</li></ul>|
|5|3/42|0/39|1/11|`data[4]` is 0 or less (this move wasn't used in the last 2 actor turns of this enemy)|
|6|3/42|3/39|3/11|`reservedata` isn't empty (there is someone available to be revived)|
|7|9/42|15/39|1/11|None, the move is always used when selected|
|8|6/42|0/39|1/11|None, the move is always used when selected|
|9|3/42|3/39|1/11|<ul><li>`data[2]` is 0 or less (this move wasn't used in the last 2 actor turns and [HoloVi](HoloVi.md) hasn't used their turn relay simulation move on this main turn)</li><li>The target of the relay is present and not IsStoppedLite (same as [IsStopped](../../Actors%20states/IsStopped.md), but [Flipped](../../Actors%20states/BattleCondition/Flipped.md) doesn't count as stopped). The target is determined randomly between [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) with 50/50 odds</li></ul>|
|10|6/42|6/39|1/11|None, the move is always used when selected|

As a sidenote, the entire move selection process occurs in the HoloKabbu coroutine who is yield returned by DoAction.

## Pre move logic
The following logic always happens before the usage of a move (this happens in DoAction, the rest happens in HoloKabbu):

- entity.`sprite`.material.renderQueue set to 2500
- CreateShield called on the entity which initialises their `bubbleshield` if it didn't exist yet

## Move 1 - Horn slash
A single target horn slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|`enemydata[actionid]`<sup>1</sup>|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>2</sup>|null|`commandsuccess`|

1: This is incorrect because `actionid` is unused so it's always 0 and under normal gameplay, this enemy should always be `enemydata[1]`. The overall affect is the attacker is incorrectly set to be [HoloVi](HoloVi.md) while `HoloKabbu` should have been the attacker here.

2: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence
This is done by yield returning the EnemyHeavyStrike coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) `playertargetentity` position + (1.5, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 103
- `Charge3` sound plays using `sounds[9]` with 1.2 pitch
- ShakeSprite called with intensity (0.25, 0.0, 0.0) and 45.0 frametimer
- Yield for 0.75 seconds
- `sounds[9]` stopped
- animstate set to 104
- `HugeHit` sound plays
- ShakeScreen called with 0.2 ammount for 0.75 time with dontreset
- DoDamage 1 call happens
- `playertargetentity`'s y `spin` set to 35.0
- `playertargetentity` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0
- Yield for 0.5 seconds
- Yield all frames until `playertargetentity` is `onground`
- `playertargetentity`'s `spin` zeroed out
- Yield for 0.5 seconds

## Move 2 - Revives [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md)
Revives [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md) from `reservedata` leaving their at 6 `hp` with canmove (so they can take an actor turn immediately on the same main turn). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Yield return ItemAnim with the battleentity with the `MiracleShake` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- `data[3]` is set to 1 which prevents this move from being used again in the battle (move 3 will be used instead if applicable, see the move selection section above for details)
- `data[0]` is incremented. If it reaches 5, this move, move 3 and move 4 can't be used anymore
- `data[1]` is set to 3 so this move, move 3 and move 4 can't be used for the next 2 actor turns
- For each `reservedata`:
    - [ReviveEnemy](../ReviveEnemy.md) called to revive the enemy party member with an hppercent of 6.0 (leaves them at 6 `hp`) with canmove and showcounter
- Yield for 1.0 second

## Move 3 - Revives another enemy party member with 7 `hp`
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

## Move 4 - [Poison](../../Actors%20states/BattleCondition/Poison.md) infliction on [HoloVi](HoloVi.md)
Inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition on [HoloVi](HoloVi.md) bypassing the `poisonres` check for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Yield return ItemAnim with the battleentity with the `DangerShroom` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on [HoloVi](HoloVi.md) to inflict them the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition for 3 main turns without effect with force (effectively for 2 main turns since the current main turn ends soon). This will bypass `poisonres` check
- `Poison` sound plays
- `PoisonEffect` particles plays at [HoloVi](HoloVi.md) position
- Yield return [ItemSpinAnim](../../Visual%20rendering/ItemSpinAnim.md) with a pos of [HoloVi](HoloVi.md) position + 1.0 in y and a sprite of the `PoisonAttacker` [medal](../../../Enums%20and%20IDs/Medal.md) icon

## Move 5 - Party wide [Taunted](../../Actors%20states/BattleCondition/Taunted.md) infliction
Inflicts the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on all player party members for 2 main turns (effectively 1 main turn since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyTaunt coroutine with the battleentity with all:

- `dontusecharge` is set to true
- animstate set to 109
- Camera moves to look near this enemy
- `Taunt2` sound plays on loop using `sounds[9]`
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- For each player party members whose `hp` is above 0:
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on the player party member for 2 main turns (effectively 1 since the main turn advances soon after)
    - `Taunt` sound plays
    - animstate of the player party member set depending on `playertargetID` (NOTE: This is incorrect because this move isn't single target so the value depends on whatever it happened to be assigned before):
        - 0 (`Bee`): 10 (`Flustered`)
        - 1 (`Beetle`): 5 (`Angry`)
        - 2 (`Moth`): 102
    - `Prefabs/Particles/AngryParticle` instantiated childed to the player party member with local position being Vector3.up
    - Yield for 0.5 seconds
    - Yield for 0.25 seconds
    - `sounds[9]` stopped
    - `Prefabs/Particles/AngryParticle` destroyed
    - the player party member animstate restored to its value before this action

Finally, `data[4]` is set to 3 so this move isn't usable for the next 2 actor turns.

## Move 6 - Revives another enemy party member with 4 `hp`
Revives another enemy party member from `reservedata` leaving them at 4 `hp` with canmove (so they can take an actor turn immediately on the same main turn). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
This is done by yield returning the EnemyPepTalk coroutine with the battleentity and targetting a random `reservedata` enemy party member:

- `dontusecharge` is set to true
- SetCamera called to move the camera without target at the selected `reservedata` with a speed of 0.05 and an offset of (0.0, 2.0, -6.0) which basically zooms in on the target
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) the selected `reservedata` + (-1.25, 0.0, -0.1)
- Yield all frames until `forcemove` is done
- `flip` set to true
- `Taunt2` sound plays on loop
- animstate set to 5 (`Angry`)
- Yield for 0.75 seconds
- `Taunt2` sound stops
- Yield return a ShakeSprite call on the selected `reservedata` with an intensity of 0.1 in x for 40.0 framtimer
- [ReviveEnemy](../ReviveEnemy.md) called to revive the target with an hppercent of 4.0 (leaves them at 4 `hp`) with canmove and showcounter
- Yield for 0.5 seconds

## Move 7 - Party wide dash attack
A party wide dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|`commandsuccess`|0.0|Vector3.zero|false|null|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence
This is done by yield returning the BeetleDash coroutine with the following parameters:

- enemyid: This enemy
- damage: 3
- animprepare: 116
- animdash: 117
- startp: This enemy's position
- speed: 29.0

This is what the coroutine effectively ends up doing:

- `playertargetID` set to -1
- Camera moves slightly to the left
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-0.5, 0.0, 0.0) at 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Charge3` sound plays using `sounds[9]` with 1.2 pitch
- animstate set to 116
- ShakeSprite called with (0.25, 0.0, 0.0) intensity and 60.0 frametimer
- Over the course of 41.0 frames, this enemy moves 2.0 to the left via a lerp
- ShakeSprite called with 0.1 intensity and 40.0 frametimer
- Yield for 41.0 frames counted by framestep
- `sounds[9]` stopped
- animstate set to 117
- `trail` set to true
- `FastWoosh` sound plays
- Over the course of 29.0 frames, this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame that the x position is less than -4.5, the following happens only once:
    - ShakeScreen called with an ammount of (0.25, 0.0, 0.0) for 0.65 time with dontreset
    - `HugeHit` sound plays
    - PartyDamage 1 call happens
- position set to (15.0, 0.0, 0.0)
- `trail` set to false
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 1 (`Walk`)
- Over the course of 61.0 frames, this enemy moves to startp via a lerp while having its animstate set to 1 (`Walk`)
- `sprite` local position reset to Vector3.zero
- `checkingdead` set to null which signals the caller that this coroutine completed

## Move 8 - Rock throw
A single target rock throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|null|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyPebbleToss coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 28 (`TossItem`)
- `Toss` sound plays
- A new sprite object is created rooted at this enemy postion + (0.5, 2.0, -0.1) using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite
- Over the course of 48.0 frames, the `MightyPeeble` sprite moves to `playertargetentity` position + (0.0, 2.0, -0.1) via a BeizierCurve3 with a ymax of 3.5. Before each frame yield, `MightyPeeble`'s z angle increses by 15.0 * framestep
- DoDamage 1 call happens
- `MightyPeeble` object is destroyed
- Yield for 0.5 seconds

## Move 9 - Decrements the `cantmove` of [HoloVi](HoloVi.md) or [HoloLeif](HoloLeif.md)
Simulate a turn relay by decrementing the `cantmove` of [HoloVi](HoloVi.md) or [HoloLeif](HoloLeif.md) which grants them an additional actor turn on this main turn. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
The target is selected randomly between [HoloVi](HoloVi.md) and [HoloLeif](HoloLeif.md). As explained in the move selection above, the target must be present and not be IsStoppedLite (same as [IsStopped](../../Actors%20states/IsStopped.md), but [Flipped](../../Actors%20states/BattleCondition/Flipped.md) doesn't count as stopped). This will assume this is the case:

- Yield return on EnemyRelay with the battleentity to the target:
    - `dontusecharge` set to true
    - This enemy animstate set to 4 (`ItemGet`)
    - The target animstate set to 2 (`Jump`)
    - This enemy's y `spin` set to 15.0
    - The target's y `spin` set to 15.0
    - The target's `cantmove` is decremented which grants them an additional actor turn on this main turn
    - `Relay` sound plays
    - Yield for 0.5 seconds
    - This enemy `spin` zeroed out
    - The target `spin` zeroed out
    - The target animstate reset to its `basestate`
    - This enemy animstate reset to its `basestate`
- `data[2]` set to 3 so this move isn't usable for the next 2 actor turns
- If the target of the move's `data` isn't yet initialised, it is initialised to 10 blank slots (even if not all of them will be used)
- The target's `data[2]` is set to 2 which prevents them from using their version of this move on the current main turn

## Move 10 - Underground strike
A single target underground strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|`enemydata[actionid]`<sup>1</sup>|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>2</sup>|null|`commandsuccess`|

1: This is incorrect because `actionid` is unused so it's always 0 and under normal gameplay, this enemy should always be `enemydata[1]`. The overall affect is the attacker is incorrectly set to be [HoloVi](HoloVi.md) while `HoloKabbu` should have been the attacker here.

2: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence
This is done by yield returning the EnemyKabbuDig coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Dig` sound plays
- animstate set to 101
- `digging` set to true
- Yield for 0.65 seconds
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) `playertargetentity` position with 2.0 multiplier
- `Digging` sound plays on loop using `sounds[9]`
- Yield all frames until `forcemove` is done
- Yield for 0.25 seconds
- ShakeScreen called with 0.1 ammount for 0.75 time with dontreset
- `sounds[9]` stopped
- DoDamage 1 call happens
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `digging` set to false
- `sprite` angles reset to Vector3.zero
- y `spin` set to 20.0
- animstate set to 119
- `sprite` scale set to `startscale`
- `DigPop` sound plays
- `DirtExplode` particles plays at `playertargetentity` position
- Over the course of 46.0 frames, this enemy moves to the value before this coroutine + Vector3.up via a BeizierCurve3 with a ymax of 0.0
- position set to the value before this coroutine
- animstate set to 13 (`BattleIdle`)
- SetAnimForce called
- `spin` zeroed out
