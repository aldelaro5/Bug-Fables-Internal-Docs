# `Pitcher`

> NOTE: This enemy is involved in the complexities of the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition. It is recommended to check the condition's documentation to learn more on how it involves this enemy.

## Assumptions
It is assumed that this enemy has its [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to the [matching EventDialogue](../../Battle%20flow/EventDialogues/Pitcher.md) as otherwise, the [Eating](../../Actors%20states/BattleCondition/Eaten.md) system can break when this enemy dies with a non null `ate`.

It is also assumed that this enemy is fought alone otherwise, his [PitcherFlytrap](PitcherFlytrap.md) summoning in the post move won't work correctly. It may also badly affect the value of `playertargetID` when deciding whether or not to eat after using the slam attack move if this enemy isn't intially fought alone.

It is also assumed this enemy is loaded with the `Pitcher` [animid](../../../Enums%20and%20IDs/AnimIDs.md) because the eating logic assumes the presence of `extra[0]` which is initialised by [CheckSpecialID](../../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) for this animid.

## [StartBattle](../../StartBattle.md) special logic
There is special logic in StartBattle that applies to this enemy after it is loaded:

- This enemy x position is increased by 1.0

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 3 element with a starting value of 0.

- `data[0]`: A cooldown for summoning a [PitcherFlytrap](PitcherFlytrap.md) in the post move. When a summon happens in the post move, the value is set to 2 (1 instead if hardmode is true). After a move is used in the post move logic, if the value is above 0, it is decremented and a [delayed projectile](../../Actors%20states/Delayed%20projectile.md) may happen instead. This also happens if there's already 3 or more `enemydata`. For the summon to be usable again when there's less than 3 `enemydata`, the value needs to be 0 or below. Essentially, it enforces a 2 actor turns (1 instead if hardmode is true) time where this enemy can't summon again
- `data[1]`: A cooldown for launching a [delayed projectile](../../Actors%20states/Delayed%20projectile.md) in the post move. When delayed projectile is launched in the post move, the value is set to a random number between 2 and 3 inclusive. If it was decided that a [PitcherFlytrap](PitcherFlytrap.md) summon won't happen in the post move, a delayed projectile will be launched, but only if this value is 0 or below. If it's not, it will be decremented instead. Essentially, it block the next 2 or 3 times to launch a delayed projectile if it otherwise would have happened
- `data[2]`: A cooldown for usage of the slam and eat move. See the section below for details as there's many caveats related to this value.
- `data[3]`: An actor turn countdown that tracks how many actor turns are left for the eaten player party member to be spit out (excludes the actor turn when the eating occurs). The value starts at 3 when this enemy eats a player party member (tracked by `ate`). At the end of the action in the post move logic except for the actor turn the eating happened, if this enemy `ate` a player party member and this value is above 0, it is decremented. If this decrement made the value 0, the player party member will be spit out. Effectively, this enforces that a player party member won't stay eaten for longer than 3 full actor turns spent by this enemy

### About `data[2]`
This value behaves in unexpected ways. It's supposed to be an antispam for the usage of the slam and eat move, but it only works partially.

The main logic this value modifies is that the slam and eat move requires this value to be 0 or below. If this isn't the case, the move cannot be used even when selected.

However, the way this value behaves extremely inconsistently and in complicated ways. Here's how it flows:

- Everytime the move is used, the value is set to 1 if [HPPercent](../../Actors%20states/HPPercent.md) is 0.7 or above. Otherwise, the value is set to 2
- In the post move logic, the value is decremented if it is above 0, but only if `ate` isn't null meaning this enemy actually ate a player party members. This means some things:
    - If the value was set to 1, it won't have any effects. This is because if it was, it wouldn't have been possible for the eating part of the move to happen so it will always be decremented to 0 which makes the slam move usable again. Essentially, if [HPPercent](../../Actors%20states/HPPercent.md) is 0.7 or above, the value does nothing
    - If the value was set to 2, but the enemy decided to not eat, then it will prevent the slam move to be used on the next actor turn only (after that actor turn, the move becomes usable again). This can only happen if only 1 player party member was alive when the move was used (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) or the selected `playerdata[playertargetID]` for eating had an `hp` of exactly 1
    - If the value was set to 2 and the enemy decided to eat, it will never get decremented until `ate` goes to null which can only happen when [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) gets called either by this enemy or their [EventDialogue](../../Battle%20flow/EventDialogues/Pitcher.md). Effectively, it completely prevents usage of the move until the player party member is spit out so in this case, the value 2 doesn't mean anything, only that it is still above 0
- When the `data[3]` countdown reaches 0 in the postmove while this enemy is still eating a player party member, `data[2]` is set to 0. This unlocks the usage of the slam move
- If [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) was called before `data[3]` countdown expires (happens when the player party member died during the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition's turn advance after sustaining the damages), `data[2]` won't change. This means several things:
    - It now means that the slam move still won't be usable for the next 2 actor turns because its value wasn't ever decremented and spitting the player party member out now allows the post move logic to decrement it again. Essentially, it will prevent the slam move to be used 2 actor turns after the player party member is spit out by the [EventDialogue](../../Battle%20flow/EventDialogues/Pitcher.md)
    - The source of the [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) call influences when the move will become usable again. If this enemy spits out, it becomes immediately usable again, but if the [EventDialogue](../../Battle%20flow/EventDialogues/Pitcher.md) did, this enemy still needs to spend 2 actor turns after the spit before being able to use the move

Here's the summary of everything above:

- If [HPPercent](../../Actors%20states/HPPercent.md) is 0.7 or above, `data[2]` does nothing
- If [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.7 and the slam move was used, but this enemy decided to not eat, the move won't be usable for the next 1 actor turn. This can only happen if only 1 player party member was alive when the move was used (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) or the selected `playerdata[playertargetID]` for eating had an `hp` of exactly 1
- If [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.7, the slam move was used and this enemy decided to eat, then the move becomes unusable until the player party member is spit out. The exact moment the move becomes usable however depends on how the [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) call happened:
    - If it happened from this enemy as a result of the `data[3]` cooldown expiring, the move immediately becomes usable once this happens
    - If it happened from the [EventDialogue](../../Battle%20flow/EventDialogues/Pitcher.md) (meaning the player party member died during the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition's turn advance after sustaining the damages), the move can only becomes usable after 2 actor turns passed

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The value `data[0]` is set to when summoning a [PitcherFlytrap](PitcherFlytrap.md) in the post move logic changes to 1 from 2. This decreases the amount of actor turn this enemy won't be able to summon again to 1 from 2
- The framespeed of each AddDelayedProjectile done in the postmove changes to 35.0 from 42.0
- In the bubbles projectiles throw move, the speed of each Projectile calls changes to 19.0 from 25.0
- In the poison breath attack move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the poison breath attack move, the damageammount of each DoDamage calls changes to 2 from 3

## Move selection
3 moves are possible:

1. A multiple targets poison bubbles projectiles throw
2. A party wide slam attack that may [Eat](../../Actors%20states/BattleCondition/Eaten.md) a player party member
3. A party wide poison breath attack

Move 2 is always used if all of the following conditions are true:

- The current [map](../../../Enums%20and%20IDs/Maps.md) is the `TestRoom` 
- `ate` is null (isn't eating anyone)
- `data[2]` is 0 or below (move 2's cooldown expired)

Otherwise, the decision of which move to use is based on odds, but the odds changes depending on if [HPPercent](../../Actors%20states/HPPercent.md) is at least 0.5 or not. Here are the odds in either situations:

|Move|Odds when [HPPercent](../../Actors%20states/HPPercent.md) >= 0.5|Odds when [HPPercent](../../Actors%20states/HPPercent.md) < 0.5|
|---:|----|----|
|1|5/11|2/10|
|2|3/11|6/10|
|3|3/11|2/10|

However, if move 2 is selected and `ate` isn't null (eating someone) or `data[2]` is above 0 (the cooldown on the usage of the move hasn't expired yet), a continue directive is issued to the enemy action loop which restarts the whole action and effectively rerolls the move selection process.

## Post move logic
The following logic always happen after using a move. It's split in 4 parts done in order: `data[2]` decrement, [PitcherFlytrap](PitcherFlytrap.md) summon, [delayed projectile](../../Actors%20states/Delayed%20projectile.md) launch and [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout).

### `data[2]` decrement
This part only happens if `data[2]` is above 0 and `ate` is null (meaning this enemy isn't currently eating a player party member)

The only thing that happens when the above is fufilled is `data[2]` is decremented. This has complex interations, see the `data` section above for details.

### [PitcherFlytrap](PitcherFlytrap.md) summon
This part only happens if `data[0]` is 0 or below (the cooldown on summoning expired) and there's less than 3 `enemydata`.

If the above isn't fufilled, `data[0]` is decremented instead.

Here's what the logic does if the above is all fufilled:

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `PitcherShake` sound plays
- animstate set to 11 (`Hurt`)
- Yield for 0.5 seconds
- `PitcherSummon` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [PitcherFlytrap](PitcherFlytrap.md) enemy with type `FromGroundKeepScale` without cantmove at the first available location among the following:
    - (5.0, 0.0, -1.15)
    - (1.2, 0.0, 0.95)
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)
- Yield all frames until `checkindead` is null (the coroutine completed)
- `data[0]` is set to 2 (1 instead if hardmode is true) which is the cooldown for being able to summon again

### [delayed projectile](../../Actors%20states/Delayed%20projectile.md) launch
This part only happens if all of the following conditions are fufilled:

- The [PitcherFlytrap](PitcherFlytrap.md) summon part didn't happen
- `data[1]` is 0 or below (the cooldown on lauching a delayed projectile expired)
- There is at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md))

If the above aren't fufilled, `data[1]` is decremented instead.

The logic sequence section below outlines what happens when all of the above are fufilled.

#### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Always happen|A new `Prefabs/Objects/PoisonBubble` GameObject rooted positioned at (-2.0, 15.0, 0.0) with a scale of 0.8x on layer 14 (`Sprite`) with its SpriteBounce's `startscale` set to true and a `PoisonEffect` particles child which has a scale of 1.25x|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|2|0|[Poison](../../Damage%20pipeline/AttackProperty.md)|42.0 (35.0 instead if hardmode is true)|This enemy|`BubbleBurst`|`PoisonEffect`|`Fall2`|

#### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `PitcherGurgle2` sound plays
- Camera moves to look near (2.75, 4.5, -0.1)
- animstate set to 103
- Yield for 0.5 seconds
- `anim` speed set to 2.0
- Yield for 0.5 seconds
- `anim` speed set to 1.0
- animstate set to 104
- `PitcherSpit2` sound plays
- A new `Prefabs/Objects/PoisonBubble` GameObject is created rooted positioned at this enemy + (-0.95, 6.5, -0.1) with a scale of 0.8x on layer 14 (`Sprite`) with its SpriteBounce's `startscale` set to true and a `PoisonEffect` particles child which has a scale of 1.25x
- MainManager.MoveTowards called to move `Prefabs/Objects/PoisonBubble` to (this enemy x position - 3.0, 15.0, 0.0) with 30.0 frametime without smooth and without local (done in paralel)
- Yield for a second
- `Prefabs/Objects/PoisonBubble` position set to (-2.0, 15.0, 0.0)
- AddDelayedProjectile 1 call happens
- `data[1]` is set to a random number between 2 and 3 inclusive (this is the cooldown for launching another delayed projectile)

### [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout)
This part happens if all of the following conditions are fufilled:

- The slam move wasn't used on this actor turn (this is to exclude the same actor turn than eating to the spit out countdown)
- `ate` isn't null (this enemy is currently eating a player party member)
- `data[3]` is above 0 (the spit out countdown hasn't yet expired)

Here's what happens if all of the above are fufilled:

- `data[3]` is decremented
- If `data[3]` just became 0 the player party member is spit out:
    - `data[2]` is set to 0 (see the `data` section above for details on what this means)
    - `spitout` is set to a new [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) call with this enemy as the eater
    - Yield for a frame
    - `gottaspit` set to false
    - Yield all frames until `spitout` is null (the coroutine completed)

## Move 1 - Poison bubbles projectiles
A multiple targets poison bubbles projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 to 3 times, but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3|[Poison](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new `Prefabs/Objects/PoisonBubble` GameObject rooted positioned at this enemy + (-4.25, 1.8, -0.1)|25.0 (19.0 instead if hardmode is true)|4.0|null|`PoisonEffect`|`BubbleBurst`|null|Vector3.zero|false|

### Logic sequence

- animstate set to 100
- `PitcherGurgle` sound plays
- Yield for 0.5 seconds
- `anim` speed set to 2.0
- Yield for 0.5 seconds
- `anim` speed set to 1.0
- The amount of hits is determined. It's random between 2 and 3 inclusive. A new GameObject array is created with this amount as the length to hold the projectiles objects
- For each hits to do, if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - animstate set to 102
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - Yield for a frame
    - `PitcherSpit` sound plays
    - The projectile element is set to a new `Prefabs/Objects/PoisonBubble` GameObject rooted positioned at this enemy + (-4.25, 1.8, -0.1)
    - Projectile 1 call happens
    - Yield for 0.5 seconds
    - animstate set to 100
    - Yield for 0.25 seconds
- Yield all frames until all projectiles objects are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 2 - Slam and [Eat](../../Actors%20states/BattleCondition/Eaten.md)
A party wide slam attack that may [Eat](../../Actors%20states/BattleCondition/Eaten.md) a player party member.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence
The logic is done in 3 parts: the slam, eating and ending.

#### Slam
This part always happen:

- `data[2]` is set to 1 unless [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.7 where it's set to 2 instead. This is the cooldown for the usage of this move. NOTE: this has several caveats, see the `data` section above for more information
- animstate set to 105
- `PitcherGurgle2` sound plays
- Yield for a second
- Camera moves to look near (-3.7, 0.0, 2.1)
- animstate set to 107
- Yield for 0.5 seconds
- ShakeScreen called with 0.2 ammount and 0.75 time
- `impactsmoke` particles plays at `partymiddle`
- PartyDamage 1 call happens
- `PitcherSpit2` sound plays
- For each player party members whose `hp` is above 0:
    - Their y `spin` is set to 20.0
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 25.0 (35.0 instead if [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.7)
    - Their animstate is set to 11 (`Hurt`)
- Yield for 46.0 frames counted by a local MainManager.`framestep` counter

#### Eating
This part always happen, but the logic changes if all of the following requirements are fufilled meaning the enemy decided to eat:

- [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.7
- There's at least 2 player party members alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))
- [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) without nullable returns a player party member whose `hp` is 2 or above

If this enemy decided to not eat, this part's logic only does the following:

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 109
- Yield for 0.5 seconds

Otherwise, if this enemy decided to eat, the full logic happens where `playerdata[playertargetID]` is eaten:

- `data[3]` is set to 3 (this is the actor turn countdown before spitting out the player party member which doesn't include the current main turn)
- If `playerdata[playertargetID]` has the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition:
    - Their `shieldenabled` is set to false
    - [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) is called to remove their [Shield](../../Actors%20states/BattleCondition/Shield.md) condition
- `playertargetentity` is set to `playerdata[playertargetID]`'s battleentity
- animstate set to 110
- Yield all frames until `playertargetentity` (who's still falling from the jump earlier) y position becomes equal or lower than this enemy's `extra[0]` (their stem's end) y position - 1.5. Before each frame yield, this enemy's middle stem bone has its position set to a lerp from the current position to (`playertargetentity` initial x position before these yields + 5.0, 4.0, `playertargetentity` z position before these yields) with a factor of 0.075 * MainManager.`framestep`
- animstate set to 111
- [Eat](../../Actors%20states/BattleCondition/Eaten.md#eat) called with this enemy as the eater and `playertargetID` as the eaten for 5 main turns (the turn counter doesn't do anything practical, see the eat documentation for details)
- `PitcherEat` sound plays
- Yield for a second
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 0 (`Idle`)
- This enemy's middle stem bone's local position is reset to its value before this action

#### Ending
This part always happens:

- Yield all frames (up to 150.0 frames) until every player party members is `onground` or their y position is 0.15 or lower

## Move 3 - Poison breath attack
A party wide poison breath attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen for each player party member whose `hp` is above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)|This enemy|The player party member|3 (2 instead if hardmode is true)|[Poison](../../Damage%20pipeline/AttackProperty.md)|null|`barfill` is 1.0 or above after DoCommand 1 (meaning the bar got filled)|

### Logic sequence

- Camera moves to look near (2.75, 1.0, 2.0)
- animstate set to 100
- `PitcherGurgle2` sound plays
- Yield for 0.5 seconds
- `anim` speed set to 2.0
- Yield for 0.5 seconds
- `anim` speed set to 1.0
- animstate set to 101
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Mist` sound plays
- `poisonsmoke` particles plays at this enemy position + (-4f, 1.8f, -0.1f) with 7.5 alivetime
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the `TappingKey` command's help)
- DoCommand 1 call happens
- All player party member whose `hp` is above 0 has their y `spin` set to 15.0
- Yield all frames until `doingaction` goes to false. Before each frame yield, all player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`) (but if `blockcooldown` is above 0 meaning a block was done, it's set to 24 (`Block`) instead)
- All player party member whose `hp` is above 0 has DoDamage 1 call happen on them followed by their `spin` getting zeroed out
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- Yield for 0.5 seconds
