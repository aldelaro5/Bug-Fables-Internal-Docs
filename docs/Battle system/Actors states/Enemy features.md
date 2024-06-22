# Enemy features
Certain [enemies](../../Enums%20and%20IDs/Enemies.md) can opt in to some features by having some of their [enemy data](../../TextAsset%20Data/Enemies%20data.md) fields setup in a certain way or simply in their [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)'s action logic. There are enough features which are complex enough that they warrant their own explanations in a separate page. These features will be documented here.

## `eventondeath`
This feature allows to trigger an [EventDialogue](../Battle%20flow/EventDialogue.md) when the enemy party member dies.

If the value isn't -1 and the enemy party member was detected dead (without `flee`) by [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) while `inevent` is false (meaning an EventDialogue wasn't in progress already), an [EventDialogue](../Battle%20flow/EventDialogue.md) whose id is the `eventondeath` will be triggered.

## `eventonfall`
This feature allows to trigger an [EventDialogue](../Battle%20flow/EventDialogue.md) when the enemy party member drops.

When set to any non negative value, whenever [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) detects that the enemy party memeber should be dropped, it won't do it as it would have normally. Instead, it will set `calleventnext` to `eventonfall` which has the effect that on the next [CheckEvent](../Battle%20flow/Update%20flows/Controlled%20flow.md), the [EventDialogue](../Battle%20flow/EventDialogue.md) whose id was `eventonfall` will be triggered.

NOTE: This feature, while fully functional is UNUSED in practice. No actual [enemy data](../../TextAsset%20Data/Enemies%20data.md#enemydata-data) entry has any non negative values.

## `moves`
The value of this field determines the base amount of actor turn an enemy is allowed to have per main turn.

In more specific terms, it means that on the [AdvanceTurnEntity](../Battle%20flow/AdvanceTurnEntity.md) of the enemy party member, unless there is a specific [condition](Conditions.md) preventing this, the `cantmove` will be set to -`moves` + 1. It means having a laoded `moves` of 1 sets the `cantmove` to 0 every main turn and if the loaded `moves` is 2, it's set to -1 and so on. Intuitively, it's the always the amount of actor turns available per main turn.

## `deathtype`
This field tells the manner in which the enemy dies on their battleentity's [Death](../../Entities/EntityControl/Notable%20methods/Death.md). It maps to a [DestroyType](../../Entities/EntityControl/Notable%20methods/Death.md#the-destroy) like so:

|deathtype|entity.destroytype|
|--------:|------------------|
|0|SpinSmoke (without `reservedata`)|
|1|SpinNoSmoke|
|2|KO|
|3|SpinSmoke (with `reservedata`)|
|4|KO|
|5|SpinKO|
|6|Shrink|
|7|ShrinkNoSmoke|
|8|None|
|9|Sink|
|10|ExplodeAnim|
|11|DropSprites|
|12|None (destroyed on the next main turn)|

However, there are some special notes to add on some of them:

- 2, 3, 4 and 5: These values supports the `reservedata` feature. This feature will have [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) kill the enemy normally, but it will also add the enemy party member to a special array called `reservedata`. This is done for 2 reasons: it prevents the enemy's object from being destroyed which allows them to still be rendered and it also allows them to be revived or accessed after their death
- 2 and 4: These 2 are functionally equivalent and share the same logic
- 12: This one is handled by [DoDamage](../Damage%20pipeline/DoDamage.md) after sustaining lethal damage and it ends by setting the battleentity's tag to `DestroyTurn` which destroys it on the next [AdvanceMainTurn](../Battle%20flow/AdvanceTurnEntity.md). It is very specific to the following [enemies](../../Enums%20and%20IDs/Enemies.md):
    - `KeyR`
    - `KeyL`
    - `Tablet`

## Difficulty scaling
This feature is documented extensively in the appropriate [section](../../TextAsset%20Data/Enemies%20data.md#stats-difficulty-scaling) of enemy data documentation because it involves a lot of loaded fields and some complex logic.

## EXP
This feature is documented extensively in the appropriate [section](../../TextAsset%20Data/Enemies%20data.md#exp-logic) of enemy data documentation because it involves a lot of loaded fields and some complex logic.

## `holditem` and `helditem`
Enemy party members have the ability to hold an [item](../../Enums%20and%20IDs/Items.md). `holditem` holds its id and `helditem` holds its SpriteRenderer.

This is primarily used for stealing items from the player party or for visually rendering them using an item. The item can be dropped by calling [DropItem](Enemy%20party%20members/DropItem.md).

## `hitaction`
This field tells if an enemy wants to performa an action immediately on the next [controlled flow](../Battle%20flow/Update%20flows/Controlled%20flow.md) update. These actions are performed out of the main turn flow because they are ran during the [player phase](../Battle%20flow/Main%20turn%20life%20cycle.md#player-phase). It's essentially a way for an enemy party member to temporarilly seize control of the turn flow to perform their action.

During [Update](../Battle%20flow/Update.md), all `enemydata` elements are checked if any has `hitaction` set to true. If none do, nothing happens. For each that does have it set to true, the following happens:

- If the enemy [IsStopped](IsStopped.md) returns true, their `hitaction` gets set to false and nothing happens as it gets skipped
- Otherwise, `enemy` is set to true and a [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) call occur with the battleentity with actionid being the `enemydata` index. This will also end the update cycle

Setting `enemy` to true here is temporary: it will go back to false as part of [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). DoAction is also responsible for setting the enemy's `hitaction` back to false. The overall effect is DoAction temporarily seize control of the battle flow, but for the game, it's as if we were in an [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase). The flow will go back to where it was in the player phase once it's handled. Due to `hitaction` being true, DoAction is able to handle this special case in a separate fashion.

This cycle repeats for all applicable enemies untill all `hitaction` are set to false at which point, the main turn procedure can continue.

There are 3 ways to automatically have `hitaction` set to true upon attack:

- `onhitaction`
- `chargeonotherenemy`
- `isdefending` being -1

Details are in the sub sections below.

### `onhitaction`
There is a standard way to have `hitaction` set automatically when the enemy party member is targetted during [DoDamage](../Damage%20pipeline/DoDamage.md) and it's done by the `onhitaction` field. The conditions required to set `hitaction` to true is that `enemy` is false (we are in the [player phase](../Battle%20flow/Main%20turn%20life%20cycle.md#player-phase)) and something else that depends on the field's value:

- If it's 1, it's always fufilled
- If it's 2, it's only fufilled if target.[position](BattlePosition.md) is `Flying`
- If it's 3, it's only fufilled if target.[position](BattlePosition.md) is `Ground`

This allows the `hitaction` to happen whenever the enemy party member is targetted while in any or in specific positions.

### `chargeonotherenemy`
There is an alternative way to get `hitaction` set to true automatically during [DoDamage](../Damage%20pipeline/DoDamage.md), but it only applies for enemy party members other than the target. That way is the `chargeonotherenemy` array field which holds an array of [Enemy](../../Enums%20and%20IDs/Enemies.md) ids.

How it works is when the target sustains the damage, every other enemy party members who has the target.`animid` (their [enemy](../../Enums%20and%20IDs/Enemies.md) id) in their `chargeonotherenemy` has their `hitaction` set to !`enemy` (false during the [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase), true during the [player phase](../Battle%20flow/Main%20turn%20life%20cycle.md#player-phase)).

### About `isdefending` being -1
There is a second way to automatically set `hitaction` to true when the enemy party member is targetted during [DoDamage](../Damage%20pipeline/DoDamage.md). That was is by having `isdefending` be -1 and the [position](BattlePosition.md) of the enemy party member must not be `Underground`. If this applies, it will override `onhitaction` and take priority by always setting `hitaction` to true.

In practice, only the `Underling` [enemy](../../Enums%20and%20IDs/Enemies.md) does this, every other Enemy defined in the game would rather use `onhitaction` or `chargeonotherenemy`.

## `defenseonhit` and `isdefending`
The value of this field has 3 modes of operations:

- -1: See the section above about `hitaction`
- 0: The enemy party member is opting out of the `hitaction` and the `isdefending` logic
- 1 or above: No `hiaction` logic, but it will get the `isdefending` logic.

As for the `isdefending`, it's where the enemy party member guards and it gain points of defenses during [DefaultDamageCalc](../Damage%20pipeline/CalculateBaseDamage.md) when piercing doesn't apply with the amount corresponding to the value and those defense points are recognised by [TrueDef](../Visual%20rendering/RefreshEnemyHP.md#truedef). They will apply when `isdefending` is true unless the enemy party member is [Flipped](BattleCondition/Flipped.md). This also enables `isdefending`'s toggling during [DoDamage](../Damage%20pipeline/DoDamage.md) (See the section below for more details). NOTE: There are several caveats with this, check the [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation to learn more.

### `isdefending`
As for `isdefending`, it is a field that tells if the enemy party member is guarding. Its initial value is false set by [StartBattle](../StartBattle.md). 

Guarding is the only way to gain the `defenseonhit` points of defenses and the main way it's done is by [DoDamage](../Damage%20pipeline/DoDamage.md) where the enemy party member is targeted. It's set to true if all of the following conditions are fufilled:

- property is't `Flip` (`Flip` prevents guarding)
- The final amount of damages calculated is above 0
- The enemy party member's [IsStopped](IsStopped.md) is false (if it's true, `isdefending` is set to false instead which breaks guard)

It's also possible to manually guard from [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)'s action logic.

The guarding is visually represented by having [UpdateAnim](../Visual%20rendering/UpdateAnim.md) set battleentity.`animstate` to 24 (`Block`) when applicable (see the method's documentation for the exact conditions required).

However, it is possible to loose the guard in many ways which sets `isdefending` to false:

- [SetCondition](Conditions%20methods/SetCondition.md): When inflicting (or amending) a [Freeze](BattleCondition/Freeze.md), [Numb](BattleCondition/Numb.md) or [Sleep](BattleCondition/Sleep.md) condition
- [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md): When hit with an attack that has a `Flip` property, guard is broken
- [ClearStatus](Conditions%20methods/ClearStatus.md): Breaking guard is part of the procedure of this method
- [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md): All enemy party members will break guard before their actions logic. It is of course possible to override this behavior inside the action logic itself

## `weight`
This field is a way to modify the visual rendering of an enemy sustaining an attack. A low value will make them jump or launch more in the air while a high value will make them launch less or not at all. Its impact are purely visual.

As a general rule, this should be between 0.0 and 100.0 where 100.0 won't launch at all and 0.0 will launch the furthest.

## `sizeonfreeze`
When the enemy party member has [Freeze](BattleCondition/Freeze.md) condition, GetEnemySize will return its `sizeonfreeze` instead of its `size` if the value is above 0.1. This is only used for visual effects and animations, but it allows to accounter for its battleentity.`icecube`.

## `data`
This is a general purpose array mostly meant for [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)'s actions logic. Any enemy party member can use it to persist informations between their actions.

The only place it is written to outside of DoAction is in [CheckSpecialID](../../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) for a `BeeBot` [animid](../../Enums%20and%20IDs/AnimIDs.md) if it's a `battle` entity. In that case, it's set to a new array of 1 element whose value is either 0 or 1 determined randomly.

## `extrastuff`
This is a general purpose transform array for use by enemy party members only during their [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)'s actions logic. Any enemy party member can use it to persist transforms between their actions.

## `weakness`
The name of this field is a missnomer because it doesn't represent only weaknesses, but rather a general purpose list of [AttackProperties](../Damage%20pipeline/AttackProperty.md) that can change the damage pipeline, especially during [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md).

## `cantfall`
When this field is true, it will never be dropped by [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) when its [position](BattlePosition.md) is `Flying`.

It will also prevent having its [position](BattlePosition.md) set to `Ground` by RefreshEnemyPos (used by [GetAvailableTargets](Targetting/GetAvaliableTargets.md) or [Chompy](../Battle%20flow/Action%20coroutines/Chompy.md)).

`lockposition` is a field that does the same logic in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) specifically, but it doesn't require the [position](BattlePosition.md) to be `Flying`. It doesn't include the other logic `cantfall` has.

## `notaunt`
When this field is true, it specifically affects the `BeetleTaunt` [skill](../../Enums%20and%20IDs/Skills.md) by preventing the [Taunted](BattleCondition/Taunted.md) infliction.

NOTE: This doesn't prevent the `forceattack` being set to `currentturn` which means that the actual enemy party members taunting logic still works despite this. This is only meant to prevent the animations or [SetCondition](Conditions%20methods/SetCondition.md) calls which is still desired for [enemies](../../Enums%20and%20IDs/Enemies.md) who do not perform any targetting actions in their action logic so it doesn't look out of place visually or logically in the case of the `TauntPlus` [medal](../../Enums%20and%20IDs/Medal.md).

## `notired`
When this is true, enemy party members will not have their `tired` (exhaustion) field incremented during [EndEnemyTurn](../Battle%20flow/EndEnemyTurn.md) like it would normally if the `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) isn't equipped.

NOTE: This applies even if [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active) because it still requires the `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) to be equipped for enemy exhaustion to be disabled.

## `hidehp`
When this is true, [RefreshEnemyHP](../Visual%20rendering/RefreshEnemyHP.md) will never render the enemy party member's `hp` and `def` stats.

## `fled`
Setting this to true causes the enemy party member to effectively die, but not in an usual way. [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) will detect the death, but process it with a very reduced logic only consisting of moving them offscreen and disabling their battleentity if their `destroyentity` is true (only happens if their `deathtype` was 12 and their `hp` got to 0 after sustaining them in [DoDamage](../Damage%20pipeline/DoDamage.md)).

## `notattle`
Setting this to true will prevent GetTattleable from including the enemy party member in the return which prevents [Tattle](../Battle%20flow/Action%20coroutines/Tattle.md) to work on it.

This is used in [SetItem](../Player%20UI/SetItem.md) when handling a [Battle strategy list type](../Player%20UI/ItemList%20confirmation%20handling/Battle%20strategy%20list%20type.md#battle-strategy-list-type-confirmation-handling) when option 1 (Spy) has been chosen.

## `actimmobile`
When this is true, the enemy party member will almost always not have their `cantmove` set to higher than 0 as a result of a stopping [condition](Conditions.md) and it will almost always have [IsStopped](IsStopped.md) return false. This means they can almost always still act while inflicted by a stopping condition.

There is one notable exception to this rule: it's possible to have IsStopped not take this field into account by sending true as the skipimmobile. This is notably done during the `hitaction` processing checks (meaning this field won't have an impact on whether an enemy party member is considered stopped for the purpose of the `hitaction`).

There are only 3 [enemies](../../Enums%20and%20IDs/Enemies.md) in the game that have this feature enabled:

- `VenusBoss`
- `WaspKing`
- `EverlastingKing`

## `diebyitself`
When all the remaining enemy party members who are alive (`hp` above 0) have this field set to true, it tells [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) to set their `hp` to 0 and their `diebyitself` to false before redoing a second (and last) enemy death checks. The second check will inevitably lead to detect their deaths and kill them properly.

The reason this happens is to handle the very specific case of killing an enemy party member while only stationary ones (such as `SandWall` or `PisciWall`) remains alive. When these cases happen, it's unecessary to keep them alive and they should die if they are the only ones that remains. In other words, each would die by itself, hence the name of the field.
