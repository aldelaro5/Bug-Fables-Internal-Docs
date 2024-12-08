# `HoloVi`

## Assumptions
This enemy is assumed to be fought with [HoloKabbu](HoloKabbu.md) and [HoloLeif](HoloLeif.md) for certain moves to be usable and for the move selection process to work correctly.

It is also loosely assumed that this enemy has [PoisonDamUp](../../Damage%20pipeline/AttackProperty.md) in their [weakness](../../Actors%20states/Enemy%20features.md#weakness). This is because [HoloKabbu](HoloKabbu.md) features a move that can inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition on this enemy which is logically intended to be paired with the damage bonuses of `PoisonDamUp`.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 3 element with a starting value of 0.

- `data[0]`: This reperesents the amount of time some specific moves were used in the battle by incrementing the value on each usage. If the value reaches 5, this enemy will no longer be able to use any moves in the list. Here are the implicated moves:
    - Revives another enemy party member from `reservedata` leaving them at 7 `hp`
    - Gives themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 3 main turns
    - Heals all enemy party members (including themselves) for 5 `hp` and gives each of them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns. NOTE: For this move specifically, `data[0]` will NOT increment, but the move still won't be used if `data[0]` reaches 5
- `data[1]`: An actor turn cooldown on the usage of the any moves managed by `data[0]`. On usage of any of these moves, the value is set to 2. The value is always decremented in the pre move logic when above 0 and the value must be 0 for any of the 3 moves managed by `data[0]` to be usable. Effectively, it's a 1 actor turn cooldown before any of the 3 moves become usable again
- `data[2]`: An actor turn cooldown on the usage of the turn relay simulation move. On every usage of the move, the value is set to 3 and it is always decremented in the pre move logic when above 0. For the move to be usable, the value needs to be 0. Effectively, it's a 2 actor turn cooldown on the move before it becomes usable again. NOTE: If [HoloKabbu](HoloKabbu.md) uses their turn relay simulation move to this enemy, this enemy's `data[2]` is set to 2 which is a 1 actor turn cooldown on the usage of this enemy's turn relay simulation move

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
The following all happens in the HoloVi coroutine:

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
9 moves are possible:

1. A single target beemerang throw
2. Revives another enemy party member from `reservedata` leaving them at 7 `hp` with canmove (so they can take an actor turn immediately on the same main turn)
3. Gives themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 3 main turns (effectively 2 main turns since the current main turn ends soon)
4. Heals all enemy party members (including themselves) for 5 `hp` and gives each of them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns
5. Simulate a turn relay by decrementing the `cantmove` of [HoloKabbu](HoloKabbu.md) or [HoloLeif](HoloLeif.md) which grants them an additional actor turn on this main turn
6. A single target heavy beemerang throw that inflicts the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition for 3 main turns (effectively 2 main turns since the current main turn ends soon), with a chance to cause recoil damages to this enemy
7. A single targets multiple hits beemerang throws
8. Gets 3 `charge` while loosing 3 `hp`
9. A single target multiple hits needles attack that may inflict the [Numb](../../Actors%20states/BattleCondition/Numb.md), [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition

Move 5 is always used if `turns` is 0 (the first main turn) and it will always relay to [HoloKabbu](HoloKabbu.md) instead being determined randomly. However, if this target doesn't meet the requirements outlined in the table below, the move selection process reverts to the normal procedure described below and this first attempt is counted as a failure.

Otherwise, the decision of which move to use is based on weigthed odds. However, these weighted odds have 3 possible distribution sets which will be numbered as follows (the first one that applies is used):

1. All of the following conditions are fufilled:
    - Both [HoloKabbu](HoloKabbu.md) and [HoloLeif](HoloLeif.md) are present
    - This enemy has no `charge`
    - This enemy doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
2. This enemy has no `charge` and doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition, but either [HoloKabbu](HoloKabbu.md) or [HoloLeif](HoloLeif.md) isn't present (assumed to be dead)
3. This enemy has at least 1 `charge` or they have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition

Also, each moves have additional requirements that if they aren't fufilled, the move won't be used and instead cause a reroll of the move selection process. However the amount of attempts to a move selection is tracked and after 5 failures to select a move, move 1 is always used.

Here are the odds for all distributions sets, their usage requirements and the step to take if those requirements aren't fufilled.

|Move|Odds set 1|Odds set 2|Odds set 3|Requirements|
|---:|----------|----------|----------|------------|
|1|14/91|7/56|1/13|None, the move is always used when selected||
|2|4/91|8/56|0/13|<ul><li>`data[0]` is less than 5 (move 2, 3 and 4 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2, 3 and 4 weren't used on the last 2 actor turns of this enemy)</li><li>`reservedata` isn't empty (there is someone available to be revived)</li></ul>|
|3|4/91|8/56|0/13|<ul><li>`data[0]` is less than 5 (move 2, 3 and 4 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2, 3 and 4 weren't used on the last 2 actor turns of this enemy)</li><li>This enemy doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition</li></ul>|
|4|6/91|12/56|0/13|<ul><li>`data[0]` is less than 5 (move 2, 3 and 4 weren't used 5 times or more)</li><li>`data[1]` is 0 or less (move 2, 3 and 4 weren't used on the last 2 actor turns of this enemy)</li><li>The average [HPPercent](../../Actors%20states/HPPercent.md) of all `enemydata` including this enemy is 0.8 or less</li></ul>|
|5|7/91|0/56|0/13|<ul><li>`data[2]` is 0 or less (this move wasn't used in the last 2 actor turns and [HoloKabbu](HoloKabbu.md) hasn't used their turn relay simulation move on this main turn)</li><li>The target of the relay is present and not IsStoppedLite (same as [IsStopped](../../Actors%20states/IsStopped.md), but [Flipped](../../Actors%20states/BattleCondition/Flipped.md) doesn't count as stopped). The target is determined randomly between [HoloKabbu](HoloKabbu.md) and [HoloLeif](HoloLeif.md) with 50/50 odds. NOTE: As explained above, there's an exception where `turns` is 0 (first main turn) where the target is always [HoloKabbu](HoloKabbu.md), but it's still possible that they don't meet these requirements which would cause a reroll like any other main turn</li></ul>|
|6|21/91|7/56|2/13|None, the move is always used when selected|
|7|14/91|0/56|7/13|None, the move is always used when selected|
|8|7/91|7/56|1/13|<ul><li>This enemy `hp` is 10 or more</li><li>This enemy has no `charge`</li></ul>|
|9|14/91|7/56|2/13|None, the move is always used when selected|

As a sidenote, the entire move selection process occurs in the HoloVi coroutine who is yield returned by DoAction.

## Pre move logic
The following logic always happens before the usage of a move (this happens in DoAction, the rest happens in HoloVi):

- entity.`sprite`.material.renderQueue set to 2500
- CreateShield called on the entity which initialises their `bubbleshield` if it didn't exist yet

## Move 1 - Simple beemerang throw
A single target beemerang throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyViRegular coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `nonphscal` set to true
- Camera moves to look near the midpoint of this enemy and `playertargetentity`
- animstate set to 103
- Yield for 0.5 seconds
- animstate set to 104
- Yield return a ShakeSprite call on the battleentity with an intensity of 0.1 in x and 20.0 frametimer
- `Woosh` sound plays on `sounds[8]` at 1.1 pitch on loop
- `Prefabs/Objects/BeerangBattle` instantiated rooted at this enemy position + 1.0 in y
- animstate set to 105
- Over the course of 40.0 frames:
    - `Prefabs/Objects/BeerangBattle` moves from its position + (0.5, 0.5, 0.0) to `playertargetentity` position + 0.75 in y via a BeizierCurve3. The parameters of the curve changes depedning if we are on the first 20.0 frames or the last 20.0 frames (it essentially travels to the target and back to where it was):
        - First 20.0 frames: Midpoint is the actual midpoint between from and destination + -5.0 in z and the factor goes from 0.0 to 1.0
        - Last 20.0 frames: Midpoint is (0.0, 0.0, 5.0) and the factor goes from 1.0 to 0.0
    - `Prefabs/Objects/BeerangBattle` angles changes like the following:
        - x: Always 80.0
        - y: Always 0.0
        - z: Decreases by 20.0 * MainManager.`framestep`
    - On the first frame after 20.0 frames, DoDamage 1 call happen and the `Prefabs/Objects/BeerangBattle` starts its second trajectory as explained above
- `Prefabs/Objects/BeerangBattle` gets destroyed
- `sounds[8]` stops playing with a delay of 0.1

## Move 2 - Revives another enemy party member with 7 `hp`
Revives another enemy party member from `reservedata` leaving them at 7 `hp` with canmove (so they can take an actor turn immediately on the same main turn). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `data[1]` is set to 2 so this move, move 3 and move 4 can't be used for the next actor turn
- `data[0]` is incremented. If it reaches 5, this move, move 3 and move 4 can't be used anymore
- Yield return ItemAnim with the battleentity with the `MagicDrops` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- [ReviveEnemy](../ReviveEnemy.md) called to revive a random `reservedata` with an hppercent of 7.0 (leaves them at 7 `hp`) with canmove and showcounter
- Yield for 1.0 second

## Move 3 - Gives themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
Gives themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 3 main turns (effectively 2 main turns since the current main turn ends soon). No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `data[1]` is set to 2 so this move, move 2 and move 4 can't be used for the next actor turn
- `data[0]` is incremented. If it reaches 5, this move, move 2 and move 4 can't be used anymore
- Yield return ItemAnim with the battleentity with the `VitalitySeed` [item](../../../Enums%20and%20IDs/Items.md):
    - animstate set to 4 (`ItemGet`)
    - `ItemHold` sound plays
    - A new sprite object gets created rooted at this enemy + its `itemoffset` using the item's icon as sprite
    - Yield for 1.0 second
    - The sprite gets destroyed
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on this enemy to give them the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 3 main turns with effect
- Yield for 1.0 second

## Move 4 - Heals all enemy party members for 5 `hp` with a [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition
Heals all enemy party members (including themselves) for 5 `hp` and gives each of them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `data[1]` is set to 2 so this move, move 2 and move 3 can't be used for the next 1 actor turn
- Yield return an EnemySharingStash call with the battleentity:
    - animstate set to 112
    - `BagRustle` sound plays
    - Yield for 0.5 seconds
    - animstate set to 109
    - `ItemHold` sound plays
    - For each `enemydata`, a new sprite object is created childed to the battleentity using a random [item](../../../Enums%20and%20IDs/Items.md) sprite each among `CrunchyLeaf`, `Mushroom` and `AphidEgg` with no angles. The positioning logic is complex and won't be detailed here for the sake of brevity, but it involves positioning the sprites to be close from each other while being far apart enough to see them
    - Yield for 0.5 seconds
    - y `spin` set to 15.0
    - Yield for 0.5 seconds
    - `spin` zeroed out
    - animstate set to 101
    - All items sprites gets rooted and their angles zeroed out
    - `Toss` sound plays
    - Over the course of 40.0 frames, each item sprites goes to their matching `enemydata` by moving towards to their position + 1.0 in y using a BeizierCurve with a ymax of 5.0
    - For each `enemydata`:
        - Their item sprite gets destroyed
        - [Heal](../../Actors%20states/Heal.md) called on them to heal 5 `hp`
        - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on them to give them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns
    - Yield for 1.0 second

## Move 5 - Decrements the `cantmove` of [HoloKabbu](HoloKabbu.md) or [HoloLeif](HoloLeif.md)
Simulate a turn relay by decrementing the `cantmove` of [HoloKabbu](HoloKabbu.md) or [HoloLeif](HoloLeif.md) which grants them an additional actor turn on this main turn. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
The target is selected randomly between [HoloKabbu](HoloKabbu.md) and [HoloLeif](HoloLeif.md) unless `turns` is 0 (first main turn) where [HoloKabbu](HoloKabbu.md) is always selected. As explained in the move selection above, the target must be present and not be IsStoppedLite (same as [IsStopped](../../Actors%20states/IsStopped.md), but [Flipped](../../Actors%20states/BattleCondition/Flipped.md) doesn't count as stopped). This will assume this is the case:

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
- `data[2]` set to 2 so this move isn't usable for the next actor turn
- If the target of the move's `data` isn't yet initialised, it is initialised to 10 blank slots (even if not all of them will be used)
- The target's `data[2]` is set to 2 which prevents them from using their version of this move on the current main turn

## Move 6 - Heavy beemerang throw with [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) infliction and a chance of recoil damages
A single target heavy beemerang throw that inflicts the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition for 3 main turns (effectively 2 main turns since the current main turn ends soon), with a chance to cause recoil damages to this enemy.

### `nonphscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|5|null|{ `NoSound` }|`commandsuccess`|
|2|Only happens after DoDamage 1 if this enemy `hp` is above 20 and a 33% RNG check passes|null|This enemy|3|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|null|false|

### Logic sequence
This is done by yield returning the EnemyHeavyThrow coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `nonphyscal` set to true
- Camera moves to look near the midpoint of this enemy and `playertargetentity`
- animstate set to 100
- `Woosh` sound plays on `sounds[8]` at 0.5 pitch on loop
- Over the course of 120.0 frames:
    - y `spin` smoothly increases towards 60.0
    - `sounds[8]` pitch changes from 0.5 to 1.2 via a lerp
    - After 36.0 frames passed, the following happens once:
        - `Charge3` sound plays
        - Yield return a ShakeSprite on the battleentity with an intensity of 0.1 in x and 2.0 frametimer
    - After 36.0 frames, each time that Time.frameCount is divisible by 10, DeathSmoke particles plays on the battleentity
- `sprite` local position is reset to Vector3.zero
- animstate set to 101
- `Prefabs/Objects/BeerangBattle` instantiated without angles and with a SpinAround component with an itself of (0.0, 0.0, 30.0)
- `sounds[8]` stops
- `Toss8` sound plays
- Over the course of 16.0 frames, `Prefabs/Objects/BeerangBattle` moves from this enemy position + 1.0 in y to the world [CenterPos](../../Actors%20states/CenterPos.md) of `playerdata[playertargetID]` via a lerp
- ShakeScreen called with an ammount of 0.4 and a time of 0.75
- DoDamage 1 call happens
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition on `playerdata[playertargetID]` for 3 main turns with effect
- `HugeHit` sound plays
- Over the course of 80.0 frames, `Prefabs/Objects/BeerangBattle` moves to this enemy position + 1.0 via a BeizierCurve3 with a ymax of 15.0
- `Prefabs/Objects/BeerangBattle` gets destroyed
- If this enemy has more than 20 `hp` and a 33% RNG check passes:
    - ShakeScreen called with an ammount of 0.1 and a time of 0.75
    - animstate set to 11 (`Hurt`) with `overrideanim` set to true
    - DoDamage 2 call happens
- Otherwise (no recoil damage)
    - `Ding2` sound plays
    - `DLGammaStep` sound plays
    - animstate set to 100
- Yield return a SlowSpinStop call on the battleentity with a spinammount of 30.0 in y and 60.0 frametime
- `spin` zeroed out
- `overrideanim` reset to false
- FlipAngle called with setangle

## Move 7 - Multiple hits beemerang throws
A single targets multiple hits beemerang throws.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|
|2_a|Happens up to 3 times (4 times if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5), but each hit requires that the previous DoDamage call dealt more than 0 damage|This enemy|The same as DoDamage 1|DoDamage 1's damageammount - `x` clamped from 1 to 99 where `x` starts at 1 on the first hit and increments after each hit|[Raw](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|2_b|Happens each time that DoDamage 2_a didn't happen due to the previous DoDamage call dealing 0 damages|This enemy|The same as DoDamage 1|0|null|null|false|

NOTE: This enemy `charge` is zeroed out after DoDamage 1.

### Logic sequence
This is done by yield returning the EnemyTornadoToss coroutine with the battleentity with a damage of 3:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint of this enemy and `playertargetentity`
- `nonphscal` set to true
- animstate set to 100
- Yield for 0.3 seconds
- `Woosh` sound plays on `sounds[9]` at 0.5 pitch 0.6 volume on loop
- Over the course of 100.0 frames:
    - y `spin` changes smoothly towards 30.0
    - `sounds[9]` pitch changes smoothly towards 1.25 clamped from 0.5 to 1.5
- `sounds[9]` stops playing with a 0.1 delay
- `Toss` sound plays
- `Woosh` sound plays on `sounds[8]` at 0.9 pitch 0.5 volume on loop
- `spin` zeroed out
- animstate set to 101
- `Prefabs/Objects/BeerangBattle` instantiated rooted at this enemy position
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Done 4 times (5 times instead if [HPPercent](../../Actors%20states/HPPercent.md) is lower than 5):
    - Over the course of 12.5 frames (40.0 frames instead on the first iteration):
        - `Prefabs/Objects/BeerangBattle` moves from its position to `playertargetentity` position + 1.0 in y via a BeizierCurve3 with a mid of `playertargetentity` position + (0.0, 3.5, -3.0)
        - On each frame, `Prefabs/Objects/BeerangBattle` z angle is set to 20.0
    - What happens here varies (the first one that applies is done):
        - First iteration: DoDamage 1 call happens followed by `charge` being set to 0
        - Second and further iterations where the last DoDamage call dealt more than 0 damage: DoDamage 2_a happens
        - Second and further iteration where the last DoDamage call dealt 0 or less damage: DoDamage 2_b happens
    - Over the course of 22.222222 frames:
        - `Prefabs/Objects/BeerangBattle` moves from its position to `playertargetentity` position + 4.0 in y via a BeizierCurve3 with a mid of `playertargetentity` position + (0.0, 3.0, 2.0)
        - On each frame, `Prefabs/Objects/BeerangBattle` z angle is set to 20.0
    - `sounds[8]` pitch increases by 0.07
- Over the course of 40.0 frames:
    - `Prefabs/Objects/BeerangBattle` moves from its position to this enemy position via a BeizierCurve3 with a ymax of 5.0
    - On each frame, `Prefabs/Objects/BeerangBattle` z angle is set to 20.0
- `sounds[8]` stops playing
- animstate set to 13 (`BattleIdle`)
- `Prefabs/Objects/BeerangBattle` gets destroyed
- If `playerdata[playertargetID]`'s `hp` is still above 0, their animstate is set to their `basestate`

## Move 8 - Gets 3 `charge` and looses 3 `hp`
Gets 3 `charge` while loosing 3 `hp`. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `Charge7` sound plays
- animstate set to 24 (`Block`)
- Yield return ShakeSprite on the battleentity with an intensity of 0.1 in x and 30.0 frametimer
- Done 3 times:
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on battleentity with type 4 (green up arrow)
    - `StatUp` sound plays with a pitch of 0.9 + (`x` + 1) * 0.1 where `x` starts at 0 and increments on each iterations
    - Yield for 0.1 seconds
    - Yield for 0.1 seconds
- `charge` set to 3
- `hp` decreased by 3

## Move 9 - Multiple hits needles attack with potential [Numb](../../Actors%20states/BattleCondition/Numb.md), [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) infliction
A single target multiple hits needles attack that may inflict the [Numb](../../Actors%20states/BattleCondition/Numb.md), [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 3 times|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (same target for each hits)|3|Random between the following (all hits uses the same property):<ul><li>[Numb](../../Damage%20pipeline/AttackProperty.md)</li><li>[Poison](../../Damage%20pipeline/AttackProperty.md)</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md)</li></ul>|null|`commandsuccess`|

### Logic sequence
This is done by yield returning the EnemyNeedlePincer coroutine with the battleentity:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 115
- `WamSoft` sound plays
- Yield for 0.65 seconds
- Camera moves to look near `playertargetentity`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + 1.5 in x
- Yield until `forcemove` is done
- Done 3 times:
    - If it's the first or third iteration, animstate is set to 121
    - Yield return a ShakeSprite call on the battleentity with an intensity of 0.1 in x and 35.0 frametimer
    - position set to `playertargetentity` + 0.65 in x
    - DoDamage 1 call happens
    - animstate set to 122 except on the second iteration where it's set to 123 instead
    - For the next 30.0 frames, this enemy position is set to a lerp from its current position to `playertargetentity` + 1.5 via a lerp with a factor of 0.2 * MainManager.`framestep`
