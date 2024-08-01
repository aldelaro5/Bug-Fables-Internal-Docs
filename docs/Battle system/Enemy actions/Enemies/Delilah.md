# `Delilah`

## Assumptions
It is assumed this enemy is fought with them and [Stratos](Stratos.md) only in the enemy party as they strongly interact together. If this assumption isn't respected, exceptions may be thrown.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 4 element with a starting value of 0.

- `data[0]`: The amount of times the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and any moves among the support items set were used (see the move selection section below for details). The value is incremented each time any of these moves are used and once it reaches 10, the usage of these moves are no longer allowed. Essentially, it's a hard limit to this enemy where they cannot use any of these moves more than 10 times combined in the entire battle
- `data[1]`: A lock on the usage of any moves from the support items move selection set (see the move selection section below for details). When the value is 1, it prevents these move to be used. The value is always reset to 0 in the post move logic. The lock is never activated by this enemy, but rather by [Statos](Stratos.md) which sets it to 1 when they use their enemy party healing move
- `data[2]`: This value does nothing. It was intended to act similarly to [Stratos](Stratos.md)'s `data[1]` which is a revive move usage deterant, but the value is never incremented even when delilah uses their equivalent move. It means this deterant never activates and they always have a 70% chance to use the move if all other conditions are satisfied instead of lowering the chances to 30% after 3 usage of the move
- `data[3]`: A cooldown on the usage of any moves among the support items set (see the move selection section below for details). When any of these move is used (except for the [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) equivalent action move), the value is set above 0. On every actor turns that these moves aren't used (except for the [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) equivalent action move where this applies), the value is decremented in the post move logic. For any of these moves to become usable again, the value must be 0 or below. Essentially, it's an actor turn antispam where the moves among the support items set can't be used again until x amount of actor turns passes where x is this value. As for the exact value set by each of these moves, it depends on the move itself:
    - The [Stratos](Stratos.md) healing move with [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) sets it to 2
    - The [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions infliction move on [Stratos](Stratos.md) sets it to 1
    - The [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) move on [Stratos](Stratos.md) sets it to 1
    - The `moreturnnextturn` increment move on [Stratos](Stratos.md) sets it to 1
    - The [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) equivalent action move doesn't set the value to anything, but it will still cause a decrement on it in the post move logic and the move is still among the ones that this value restricts the usage of
    - The enemy party healing move sets it to 2
    - The [Stratos](Stratos.md) reviving move sets it to 2

## Move selection
11 moves are possible:

1. A multiple targets bazooka projectiles throws in multiple sets that have varying effects
2. A single target multiple hits dart throws that may inflict [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md)
3. A single target dash attack followed by an `hp` draining attack
4. A single target attack involving the same effects as the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) action where the target is a player party member
5. Heals [Stratos](Stratos.md) by 6 `hp` and gives them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns
6. Gives [Stratos](Stratos.md) the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions for 3 main turns (effectively 2 since the current main turn gets advanced soon)
7. Cures [Stratos](Stratos.md)'s negative conditions with [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md)
8. Increments [Stratos](Stratos.md)'s `moreturnnextturn` giving them an additional actor turn on the next main turn
9. Does the equivalent of the [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) action which clears every actor's conditions via [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md)
10. Heals 15 `hp` and calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Stratos](Stratos.md)
11. Revives [Stratos](Stratos.md) leaving them at 7 `hp`

Moves 5 through 11 belongs in a special sets of move that have their own move selection process. Since all of them involves simulating the usage of an [items](../../../Enums%20and%20IDs/Items.md) without doing damages, the move selection for them will be refered to as the support items move selection while the other moves will be refered to as the standard move selection.

### Support items move selection
To have a chance to use move 5 through 11, all of the following conditions must be fufilled:

- `data[1]` is 0 ([Stratos](Stratos.md) didn't set a lock on this enemy using these moves which happens when they used their enemy party healing move)
- This enemy doesn't have the [Sticky](../../Actors%20states/BattleCondition/Sticky.md) condition
- `data[0]` is less than 10 (this enemy hasn't reached its limit of using these moves and move 4)
- `turns` is above 0 (at least 1 main turns passed since the start of the battle)
- `data[3]` is 0 or below (the cooldown for being able to use these moves expired)

If all of the above are fufilled, it is now possible that move 5 through 11 is used. However, each must go through a complex selection process to see if a move will be used among them. It is still possible that none of these moves are used. If this happens, this enemy will falback to the standard move selection process.

Move 11 is checked first and it will be used if all of the following conditions are fufilled:

- [Stratos](Stratos.md) isn't present in `enemydata` (they died)
- A 70% RNG check passes. NOTE: there is dead logic where the odds could have changed to 30% instead if `data[2]` is at least 3, but `data[2]` is always 0 meaning the odds always are 70% (see the `data` section as to what `data[2]` was supposed to do)

If move 11 was decided to not be used, move 5 through 10 can only be used if [Stratos](Stratos.md) is still present in `enemydata`. If that's the case, this move selection process continues, but if it's not, this enemy falbacks to the standard move selection process.

If we continue, move 10 is checked next. It's only used if all of the following conditions are fufilled:

- Both this enemy and [Delilah](Delilah.md) have an [HPPercent](../../Actors%20states/HPPercent.md) lower than 0.3
- A 35% RNG check passes

If neither move 10 or 11 are used, move 5 through 9 are considered next, but only if a 37.5% RNG check passes. If it fails, this enemy falbacks to the standard move selection process.

If it passes, the decision of which moves to use among 5 through 9 is based on odds, but each have additional requirements that must be fufilled once the move is selected for the move to be actually used. If the move is selected, but the requirements aren't fufilled, this enemy falbacks to the standard move selection process. Here is a table with all these informations for each of these moves:

|Move|Odds|Requirements|
|---:|----|-----------|
|5|4/18|[Stratos](Stratos.md)'s [HPPercent](../../Actors%20states/HPPercent.md#hppercent) is below 0.75|
|6|3/18|Both of the following has to be true:<ul><li>[Stratos](Stratos.md) doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition</li><li>[Stratos](Stratos.md) doesn't already have the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition</li></ul>|
|7|5/18|At least 1 of the following has to be true:<ul><li>[Stratos](Stratos.md) is [stopped](../../Actors%20states/IsStopped.md)</li><li>[Stratos](Stratos.md) has the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition</li><li>[Stratos](Stratos.md) has the [Fire](../../Actors%20states/BattleCondition/Fire.md) condition</li></ul>|
|8|3/18|[Stratos](Stratos.md)'s `moves` is 1 (meaning they haven't yet gone through their phase transition)|
|9|3/18|PartyIsBuffed with a cutoff of 3 returns true (see the section below for details, this is logically broken and has several caveats)|

Move 9's requirements is special since it involves a method specifically for this move, but the method is logically broken.

#### About move 9 and PartyIsBuffed
Move 9 affects every actors in the battle by removing a lot of conditions and effects that are either positive or negative to them. It means that this enemy shouldn't want to use this move if it would negatively impact the enemy party more than benefit from it.

The method PartyIsBuffed was supposed to the determine if this is the case. It attempts to use multiple heuritstics to see if the player party has more positive effects (such as conditions or `charge`) than the enemy party.

```cs
private bool PartyIsBuffed(int cutoff)
```

Its intent was that it would use a number starting at 0 and perform the following on it for each positive or negative effects applied on each actor:

1. Increase the number for each positive effect applied on each player party member
2. Decrease the number for each negative effect applied on each player party member
3. Increase the number for each negative effect applies on each enemy party member
4. Decrease the number for each positive effect applies on each enemy party member

Once all this is done, the method would have returned true if the final number is higher than the cutoff parameter (here, it's always 3) and it would have returned false otherwise. This was the likely intended logic: it would tell if there's slighly more positive effects from performing move 9 or if there isn't enough reasons to use it.

However, step 3 and 4 aren't done correctly due to a programming error. Instead of counting each effect on enemy party members, it counts the effects applied on the `playerdata` who shares an `enemydata` index. Effectively, it means that:

- `playerdata[0]` (should be `Bee`)'s effects will be counted instead of `enemydata[0]` (should be [Stratos](Stratos.md))
- `playerdata[1]` (should be `Beetle`)'s effects will be counted instead of `enemydata[1]` (should be this enemy)

The only exception to this are the enemy party member's `charge` where these are counted correctly.

This drastically complexify what this method ACTUALLY does, but the end result is still the same: the final number is compared to the cutoff and the return is true if it's higher or false if it's not. It means however that the vast majority of the enemy party's effects aren't counted correctly which affects the logic. For the most part, it means that for a typical party under normal gameplay (`Bee`, `Beetle` and `Moth` in that order), `Moth`'s effects are the only ones that gets counted on the player party side since `Bee` and `Beetle` gets counted twice, but with the opposite effects so they get cancelled. It also means that except for `charge`, no effects is actually counted for the enemy party members.

The following table can be used to determine the final result considering all of this broken logic. A note about conditions check however: since when this method is invoked, the current main turn ends very soon, these checks involves the main turn counter on the [condition](../../Actors%20states/Conditions.md) being higher than 1. This is because if it was 1, it would expire very soon which means the condition isn't very useful to consider here.

As a reminder, the initial value of the result is 0 and the cutoff passed is 3 under normal gameplay:

|Condition|Effect on result|
|---------|---------------|
|`playerdata[2]` (should be `Moth`) has the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|+ 1|
|`playerdata[2]` (should be `Moth`) has the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|+ 1|
|A player party member has a `charge` above 0|+ (`charge` + 1) applied for each applicable player party members|
|An enemy party member has a `charge` above 0|- (`charge` + 1) applied for each applicable enemy party members|
|A player party member has the [Inked](../../Actors%20states/BattleCondition/Inked.md) condition with more than 1 main turn left on it|-2 applied for each applicable player party member|
|`playerdata[2]` (should be `Moth`) has the [Numb](../../Actors%20states/BattleCondition/Numb.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|- 1|
|`playerdata[2]` (should be `Moth`) has the [Freeze](../../Actors%20states/BattleCondition/Freeze.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|- 1|
|`playerdata[2]` (should be `Moth`) has the [Fire](../../Actors%20states/BattleCondition/Fire.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|- 2|
|`playerdata[2]` (should be `Moth`) has the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|- 1|
|A player party member has the [Sticky](../../Actors%20states/BattleCondition/Sticky.md) condition with more than 1 main turn left on it|-2 applied for each applicable player party member|
|`playerdata[2]` (should be `Moth`) has the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|+ 1|
|`playerdata[2]` (should be `Moth`) has the [GradualTP](../../Actors%20states/BattleCondition/GradualTP.md) condition with more than 1 main turn left on it (`Bee` and `Beetle` having it doesn't count)|+ 1|

### Standard move selection
This move selection applies to move 1 through 4 and it happens if it was determined that the support items move selection won't apply for any reasons.

The decision of which moves to use is based on odds, but those odds changes depending on if this enemy has the [Inked](../../Actors%20states/BattleCondition/Inked.md) condition or not. Here are the odds in either situations:

|Move|Odds when not [Inked](../../Actors%20states/BattleCondition/Inked.md)|Odds when [Inked](../../Actors%20states/BattleCondition/Inked.md)|
|---:|------|------|
|1|4/13|Never used|
|2|3/13|Never used|
|3|5/13|1/2|
|4|1/13|1/2|

However, move 4 has additional requirements that must all be fufilled for the move to actually be used once selected:

- This enemy doesn't have the [Sticky](../../Actors%20states/BattleCondition/Sticky.md) condition
- `turns` is above 0 (at least 1 main turn passed since the start of the battle)
- `data[0]` is below 10 (this enemy hasn't yet reached its limit on the usage of this move and any amoing the support items moves)

If the move is selected, but the requirements above aren't fufilled, another move is used instead depending on if this enemy has the [Inked](../../Actors%20states/BattleCondition/Inked.md) condition:

- If it has it, move 3 is used
- If if doesn't have it, move 1 is used

## Pre move logic
The following logic is always done before the usage of a move if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and `moves` is 1 (meaning this phase transition hasn't happened yet).

When the above is fufilled, this enemy goes into a phase transition where they get 2 actor turns per main turn including the current one:

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
The following logic is always done after the usage of a move:

- If the move that was used belongs in a certain set, `data[3]` is decremented (the cooldown for the usage of the moves used in the support items move selection). The set of moves this applies are the following:
    - Bazooka projectiles move
    - Darts throws move
    - Dash attack and `hp` drain move
    - [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move
    - [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) equivalent action move (this move belongs in the support items move selection, but it will still cause the decrement to happen instead of setting the value above 0 like other moves from the support items move selection)
- `data[1]` is set to 0 (clears the lock set by [Stratos](Stratos.md) on moves used in the support items move selection)

## Move 1 - Bazooka projectiles throws
A multiple targets bazooka projectiles throws in multiple sets that have varying effects.

This move throws projectils in 3 or 4 sets. Each set has 1 or 3 hits depending on the set itself. Each set has a unique property and amount of hits to do with other additional changes such as the base damage amount and other visual changes. See the information on the Projectile 1 call below to learn their selection odds and their specific effects.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 1 or 3 times per set depending on the set selected (it's 1 time except if the set has the `Ink` or `Sticky` property where it's 3 times) The amount of sets is random between 3 and 4 inclusive. Each sets requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)). If the entire player party becomes dead during the set, the calls still happens, but the target doesn't change between calls until the end of the set where no more calls will happen|Depends on the property of the set:<ul><li>null: 5</li><li>`Ink`: 3</li><li>`Sticky`: 3</li><li>`Freeze`: 4</li><li>`Poison`: 4</li><li>`Fire`: 5</li></ul>|The set determines the value which is determined randomly among the following weighthed odds:<ul><li>null: 2/15</li><li>`Ink`: 4/15</li><li>`Sticky`: 3/15</li><li>`Freeze`: 2/15</li><li>`Poison`: 2/15</li><li>`Fire`: 2/15</li></ul>|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls except when the player party died during the set)|A new sprite object rooted positioned at startp + (-1.0, 2.2, -0.1). The sprite, its scale and color depends on the property of the set:<ul><li>null: `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite with pure white color and 1.0x scale</li><li>`Ink`: `Sprites/Particles/bigbubble` sprite with 000033 color (dark blue) and 0.4x scale</li><li>`Sticky`: `Sprites/Particles/bigbubble` sprite with pure white color and 0.4x scale</li><li>`Freeze`: `Ice` [item](../../../Enums%20and%20IDs/Items.md) sprite with pure white color and 1.0x scale</li><li>`Poison`: `PoisonSpud` [item](../../../Enums%20and%20IDs/Items.md) sprite with pure white color and 1.0x scale</li><li>`Fire`: `FlameRock` [item](../../../Enums%20and%20IDs/Items.md) sprite with pure white color and 1.0x scale</li></ul>|Random between 35 and 44 inclusive unless property is `Ink` or `Sticky` where it's random between 25 and 34 inclusive instead|5.0 unless property is `Ink` or `Sticky` where it's random between 6.0 and 8.0 instead|`keepcolor`|Depends on the property of the set:<ul><li>null: null</li><li>`Ink`: `InkGet`</li><li>`Sticky`: `StickyGet`</li><li>`Freeze`: `mothicenormal`</li><li>`Poison`: `poisoneffect`</li><li>`Fire`: `Fire`</li></ul>|Depends on the property of the set:<ul><li>null: null</li><li>`Ink`: `BubbleBurst`</li><li>`Sticky`: `BubbleBurst`</li><li>`Freeze`: `OverworldIce`</li><li>`Poison`: null</li><li>`Fire`: null</li></ul>|null|(0.0, 0.0, 20.0)|false|

### Logic sequence

- animstate set to 107
- `WaspKingAxeThrowCatch` sound plays with 0.9 pitch
- Yield for a second
- The amount of sets of hits to do is determined. It's random between 3 and 4 inclusive. A new SpriteRenderer array is created with this amount as the length to hold the projectiles objects (NOTE: this is technically wrong because it's the amount of sets and not the amount of hits per set, but it's guaranteed to contain enough anyway so it's safe still)
- For each sets of hits to do, if there's at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
    - The set is determined. They will be identified by the property they will have on the Projectile 1 call and the amounts of hits in the set. The set is determined randomly with weighted odds:
        - null (1 hit): 2/15
        - `Ink` (3 hits): 4/15
        - `Sticky` (3 hits): 3/15
        - `Freeze` (1 hit): 2/15
        - `Poison` (1 hit): 2/15
        - `Fire` (1 hit): 2/15
    - `b33BotInstantMissle` sound plays with 1.1 pitch
    - For each hit in the set:
        - If this isn't the first hit (and only) hit, `b33BotInstantMissle` sound plays with a pitch of 1.325 if it's the second hit or 1.4 if it's the third hit
        - The projectile element is set to a new sprite object as described in the Projectile 1 call above depending on the property of the set
        - Projectile 1 call happens with the determined set
        - MainManager.MoveTowards called to move this enemy from startp + 0.5 in x to startp with 20.0 frametime without smooth and without caller (done in paralel)
        - DeathSmoke particles plays at the projectile sprite position
        - If this isn't the last hit of the set, animstate is set to 110. Otherwise, it's set to 109
        - Yield for a random interval betweem 0.65 and 1.0 seconds if there's only 1 hit in the set or between 0.3 and 0.5 seconds if there's more than 1 hit in the set
        - If there's still at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)), [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) is called
    - If this isn't the last set, animstate is set to 108
    - Yield for a frame
- Yield all frames until all projectiles elements are null (all Projectile 1 calls completed)

## Move 2 - Darts throws with potential [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) infliction
A single target multiple hits dart throws that may inflict [Poison](../../Actors%20states/BattleCondition/Poison.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times (4 times instead if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.65)|3|[Poison](../../Damage%20pipeline/AttackProperty.md) or [Sleep](../../Damage%20pipeline/AttackProperty.md) determined randomly with uniform odds|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target is the same for each calls)|A new sprite object rooted using the `PoisonDart` [item](../../../Enums%20and%20IDs/Items.md) sprite if property is `Poison` or the `NumbDart` [item](../../../Enums%20and%20IDs/Items.md) sprite if property is `Sleep` positioned at this enemy + (-0.5, 1.25, -0.1) with a z angle of -15.0|20.0|0.0|null|null|null|null|Vector3.zero|false|

### Logic sequence

- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.3, 2, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near (-1.75, 0.0, 1.6)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (2.25, 0.0, -0.5) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 103
- Yield for 0.5 seconds
- `Jump` sound plays
- `Spin2` sound plays on loop using `sounds[5]` with 1.2 pitch
- animstate set to 102
- y `spin` set to 20.0
- MainManager.MoveTowards called on this enemy to move them to (1.35, 2.5, -0.5) with 40.0 frametime without smooth and without local (done in paralel)
- Yield 0.5 seconds
- Yield 0.25 seconds
- `spin` zeroed out
- `sprite` angle set to FlipAngle
- `sounds[5]` stopped
- The amount of hits is determined. It's 2 unless [HPPercent](../../Actors%20states/HPPercent.md) is below 0.65 where it's 4 instead. A new SpriteRenderer array is created with this amount as the length to hold the projectiles objects
- For each hit to do:
    - `Toss` sound plays
    - `Spin10` sound plays
    - The property of the Projectile 1 call is determined randomly with uniform odds between [Poison](../../Damage%20pipeline/AttackProperty.md) and [Sleep](../../Damage%20pipeline/AttackProperty.md)
    - animstate set to 111
    - The projectile element is set to a new sprite object rooted using the `PoisonDart` [item](../../../Enums%20and%20IDs/Items.md) sprite if property is `Poison` or the `NumbDart` [item](../../../Enums%20and%20IDs/Items.md) sprite if property is `Sleep` positioned at this enemy + (-0.5, 1.25, -0.1) with a z angle of -15.0
    - Projectile 1 call happens
    - If this isn't the last hit:
        - Yield for 0.5 seconds

## Move 3 - Dash attack followed by `hp` drain attack
A single target dash attack followed by an `hp` draining attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|
|2|Always happen after DoDamage 1|This enemy|The same target as DoDamage 1|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near (-1.75, 0.0, 1.6)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (2.25, 0.0, -0.5) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 103
- Yield for 0.5 seconds
- `Jump` sound plays
- `Spin2` sound plays on loop using `sounds[5]` with 1.2 pitch
- animstate set to 102
- y `spin` set to 20.0
- MainManager.MoveTowards called on this enemy to move them to (1.35, 2.5, -0.5) with 40.0 frametime without smooth and without local (done in paralel)
- Yield 0.5 seconds
- Yield 0.25 seconds
- `spin` zeroed out
- `sprite` angle set to FlipAngle
- `sounds[5]` stopped
- `trail` set to true
- animstate set to 104
- Camera moves to look near `playerdata[playertargetID]`
- `FastWoosh` sound plays
- Over the course of 21.0 frames, this enemy moves to `playerdata[playertargetID]` position + (0.75, 0.75, -0.1) via a lerp
- `trail` set to false
- This enemy position set to `playerdata[playertargetID]` position + (0.35, 0.85, -0.1)
- DoDamage 1 call happens
- `sprite` gets a SpriteBounce added on it with a MessageBounce using a 1.0 multiplier and a `speed` of 15.0
- animstate set to 105
- Done 3 times:
    - `Kiss` sound plays with 1.1 pitch
    - Yield for 0.33 seconds
- DoDamage 2 call happens
- The SpriteBounce of the `sprite` gets destroyed
- `sprite` scale reset to 1.0x
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by `lastdamage` amount of `hp`
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 106
- z `spin` set to -15.0
- `trail` set to true
- `Spin3` sound plays with 0.9 pitch
- Over the course of 41.0 frames, this enemy moves to startp + 0.25 in y via a BeizierCurve3 with a ymax of 8.0
- `trail` set to false
- `spin` zeroed out
- `sprite` angles reset to Vector3.zero
- animstate set to 0 (`Idle`)
- SetAnimForce called

## Move 4 - [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action
A single target attack involving the same effects as the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) action where the target is a player party member.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charge` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action). This doesn't effectively do anything because this enemy cannot get `charge` normally. 

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|null|`playerdata[playertargetID]` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|6|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|empty array|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- `data[0]` is incremented (amount of times this this move and the support items moves were used)
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `LonglegSummoner` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- The `LonglegSummoner` sprite gets destroyed
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- `checkingdead` is set to a new LongLeg coroutine with `playerdata[playertargetID]` as the target from this enemy which will do similar logic than the player action version
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine ends up doing:

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

## Move 5 - Heals [Stratos](Stratos.md) by 6 `hp` with [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md)
Heals [Stratos](Stratos.md) by 6 `hp` and gives them the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition for 3 main turns. No damages are dealt.

### Logic sequence

- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `MiteBurg` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called to heal `Stratos` by 6 `hp`
- Yield for 0.5 seconds
- `Heal3` sound plays
- `MagicUp` particles plays at `Stratos`'s position
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on `Stratos`
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition on `Stratos` for 3 main turns
- `data[3]` is set to 2 (the cooldown on the usage of this move and other moves from the support items set)
- The `MiteBurg` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)

## Move 6 - Gives [Stratos](Stratos.md) the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions
Gives [Stratos](Stratos.md) the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions for 3 main turns (effectively 2 since the current main turn gets advanced soon). No damages are dealt.

### Logic sequence

- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `BerryShake` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on `Stratos` for 3 main turns (effectively 2 since the current main turn advances soon) with effect
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on `Stratos` for 3 main turns (effectively 2 since the current main turn advances soon) with effect
- `data[3]` is set to 1 (the cooldown on the usage of this move and other moves from the support items set)
- The `BerryShake` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)

## Move 7 - [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on [Stratos](Stratos.md)
Cures [Stratos](Stratos.md)'s negative conditions with [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md). No damages are dealt.

### Logic sequence

- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `ClearWater` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- `Heal3` sound plays
- `MagicUp` particles plays at `Stratos`'s position
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on `Stratos`
- `data[3]` is set to 1 (the cooldown on the usage of this move and other moves from the support items set)
- The `ClearWater` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)

## Move 8 - Increments [Stratos](Stratos.md)'s `moreturnnextturn`
Increments [Stratos](Stratos.md)'s `moreturnnextturn` giving them an additional actor turn on the next main turn. No damages are dealt.

### Logic sequence

- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `HustleSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- `Heal3` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `Stratos` with type 5 (yellow up arrow)
- `Stratos`'s `moreturnnextturn` incremented (this will give them an additional actor turn on the next main turn)
- `data[3]` is set to 1 (the cooldown on the usage of this move and other moves from the support items set)
- The `HustleSeed` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)

## Move 9 - [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) equivalent action
Does the equivalent of the [ClearBomb](../../Player%20actions/Items%20usage/ClearBomb.md) action which clears every actor's conditions via [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md). No damages are dealt.

### Logic sequence

- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `ClearBomb` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- The `ClearBomb` sprite gets destroyed
- `Toss` sound plays
- `Toss4` sound plays
- A new sprite object is created rooted using the `ClearBomb` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their (-0.5, 2.0, -0.1)
- Over the course of 51.0 frames, the `ClearBomb` sprite moves to Vector3.zero via a BeizierCurve3 with a ymax of 7.0. Before each frame yield, the z angle of the sprite increases by 15.0 * MainManager.`framestep`
- The `ClearBomb` sprite gets destroyed
- `checkingdead` is set to a new ClearBombEffect coroutine which will do the same logic than the player action version
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine ends up doing:

- `IceBreak` sound plays at 0.6 volume
- `impactsmoke` particles plays at Vector3.zero
- Yield for 0.3 seconds
- `impactsmoke` particles plays at (-3.0, 0.0, 0.0)
- Yield for 0.3 seconds
- `impactsmoke` particles plays at (3.0, 0.0, 0.0)
- Yield for 0.3 seconds
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on each enemy party members
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on each player party members
- `checkingdead` is set to null to signal the caller that the coroutine is done

## Move 10 - Heals 15 `hp` and [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Stratos](Stratos.md)
Heals 15 `hp` and calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on themselves and [Stratos](Stratos.md). No damages are dealt.

### Logic sequence

- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `KingDinner` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by 15 `hp`
- [Heal](../../Actors%20states/Heal.md) called to heal [Stratos](Stratos.md) by 15 `hp`
- The `KingDinner` sprite gets destroyed
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on this enemy
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on `Stratos`
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[3]` is set to 2 (the cooldown on the usage of this move and other moves from the support items set)

## Move 11 - Revives [Stratos](Stratos.md) with 7 `hp`
Revives [Stratos](Stratos.md) leaving them at 7 `hp`. No damages are dealt.

### Logic sequence

- `data[0]` is incremented (amount of times this this move, the [LongLegSummoner](../../Player%20actions/Items%20usage/LonglegSummoner.md) equivalent action move and other support items moves one were used)
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted using the `MagicDrops` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + their `itemoffset`
- Yield for a second
- [ReviveEnemy](../ReviveEnemy.md) called with `reservedata` index 0 (`Stratos`) leaving them with 0.3% of `hp` with canmove
- Yield for a frame
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP counter) with an ammount of 7 starting from `enemydata[0]`'s position (should be `Stratos`) + 1.0 in y and ending to Vector3.up
- `enemydata[0]` (`Stratos`)'s `hp` is set to 7
- `Heal` sound plays
- `enemydata[0]` (`Stratos`) animstate set to 13 (`BattleIdle`)
- `Stratos`'s `data[3]` is set to 1 which activates the lock on `Stratos`'s usage of their reviving move and enemy party healing move
- The `MagicDrops` sprite gets destroyed
- This enemy animstate set to the local startstate
- Yield for 0.5 seconds
- `data[3]` is set to 2 (the cooldown on the usage of this move and other moves from the support items set)
