# DoAction
This action coroutine allows an actor to perform an action which is an hardcoded procedure in battle such as an attack, skill or special item usage. This coroutine contains the logic of all player and enemy actions making it the largest method in the game by far in terms of code volume. This has performance implications (see the section below for details).

The vast majority of the logic of this coroutine are contained in the individual actions, but this page will focus on documenting the surrounding code as the actions will be documented somewhere else.

TODO: Document all player and enemy actions in part 3 of the battle docs

```cs
private IEnumerator DoAction(EntityControl entity, int actionid)
```

## Parameters

- `entity`: The actor that will be performing the action
- `actionid`: The meaning changes depending on the entity:
    - If it's a player party member, this is a number that tells what action entity will be performing (-2 is a `LonglegSummoner` [item](../../../Enums%20and%20IDs/Items.md) -1 is a basic attack, anything of 0 or above is a [Skill](../../../Enums%20and%20IDs/Skills.md) id or [item](../../../Enums%20and%20IDs/Items.md) use)
    - If it's an enemy party member, this is its `enemydata` index. Sending an invalid index is considered invalid and will result in an exception being thrown in the vast majority of cases
    - If the value is -555, this is reserved to perform a blank dummy call, see the section below about performance for more details

## Performance implications
This coroutine is extremely large in IL code size. This creates a problem because when calling it for the first time, it requires the JIT's full attention to compile it which can stutter the game. This performance penalty goes away once it has been called at least once throughout the entire game's session, but it is still very noticeable (it can be as long as a second or two).

One mitigation the game does is to precipitate the stutter during the battle out loading transition of [StartBattle](../../StartBattle.md). It does this by checking a boolean that belongs to MainManager and is only set to true when this happened for the first time since the last save load. To prevent any destructive logic to happen, StartBattle will send -555 as the actionid which can be seen as a blank dummy call: it will not perform any meaningful logic, but it will still be enough to force the JIT to compile the coroutine as it was explicitly called.

## Procedure
Considering how large this method its lifecycle will be divided into different phases.

NOTE: If this is a player action, the coroutine expects `availabletargets`, `target`, [currentaction](../../Player%20UI/Pick.md), [itemarea](../../Player%20UI/AttackArea.md) and `currentturn` to be correctly set because they are needed to tell the coroutine the details of the action (notably, `currentturn` tells the `playerdata` index of the player party member performing the action). For a [Skill](../../../Enums%20and%20IDs/Skills.md), its cost is expected to be in [flagvar](../../../Flags%20arrays/flagvar.md) 0 (TP if it's 0 or above, HP if it's negative using the absolute value).

### Setup
This is the first phase of the coroutine. It ensures the coroutin can proceed and does some preparations logic.

- `deadmembers` is set to the return of GetDeadParty which are all the player party members indexes whose `hp` is 0 or below
- If `cancelupdate` is true (which means we had entered into a [terminal flow](../Update%20flows/Terminal%20flow.md)), a frame is yielded followed by the coroutine abruptly ending with a yield break. This can be considered a safeguard in case terminal flow was entered before a DoAction call occured
- `overridechallengeblock` is set to false
- Any string invocation of [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is cancelled. This can happen if [AdvanceMainTurn](AdvanceMainTurn.md) invoked it, but a DoAction call processed before the incation occurs which can interfere because DoAction needs tighther control over calling UpdateAnim (and it calls it later as part of the setup anyway). Cancelling the invocation prevents this from happening
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- `action` is set to true switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)

What follows is a bunch of field value resets:

|Field|Value|
|----:|-----|
|`infinitecommand`|false|
|`hasblocked`|false|
|`dontusecharge`|false|
|`nolifesteal`|false|
|`caninputcooldown`|0.0|
|`blockcooldown`|0.0|
|`combo`|1|
|`barfill`|0.0|
|`weakenemyfound`|false|
|`killinput`|false|
|`nonphysical`|false|
|`commandsuccess`|false|

From there, a couple of local variables are initialised that may be used in the actions logic:

- flip: The `flip` value of the entity, used for restoring later in normal post action
- randomposafter: A boolean starting at false that when true by the normal post action, the `position` of the enemy party member will be determined randomly. This only applies to enemy party members and is only used by `Mushroom` and `BeeBot` under normal gameplay. NOTE: it is invalid to set this to true during a player action and if done, it will lead to illogical behaviors
- fled: A boolean starting at false that when true by the normal post action, it will change the majority of the normal post action logic to be very reduced. This is supposed to signal that the action lead to the enemy party member fleeing the battle. This only applies to enemy party members and is only used by `Burglar` and `GoldenSeedling` under normal gameplay. NOTE: it is invalid to set this to true during a player action and if done, it will lead to illogical behaviors
- nocharm: A boolean starting at false that when true by the normal post action, it will prevent any [UseCharm](../UseCharm.md) calls to occur. This means no `HealTP` for the player party if it spent any to perform a skill and no `HealHP` after an enemy action. This is only used by `WaspHealer`, `LeafbugNinja` and `LeafbugArcher` under normal gameplay
- startp: The x/y starting position of the entity (the y component is left at 0.0). This can be overriden to another value in action logic and it is only used to move the actor back to it in normal post action
- startstate: The starting [animstate](../../../Entities/EntityControl/Animations/animstate.md) of the entity. This can be overriden in action logic and is only used as the stopping state when moving the actor back to startp in normal post action
- usedtp: An integer starting at -1 that reports the cost of the skill used by the player party member if applicable. This is only used in normal post action to determine if a [UseCharm](../UseCharm.md) call should occur with `HealTP` corresponding to half of the skill's cost. This has no effect on enemy party members

After, the following is done:

- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called on the entity to unlock its `rigid`

From there, the next phase depends on the entity itself. It can either be a player action or an enemy action:

- If the entity has a `Player` tag, it's a player action
- Otherwise, this is an enemy action, but there is a special case in which it will not be performed and the coroutine will skip to the post action phase if all of the following are true:
    - `firststrike` is true
    - The `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is not equipped
    - [flags](../../../Flags%20arrays/flags.md) 614 is false (HARDEST is inactive)
    - `actionid` is not 0 (this isn't the first enemy party member)

NOTE: This enemy action exception never applies under normal gameplay. This is because `firststrike` is only true in the case where the enemy had the advantage when [StartBattle](../../StartBattle.md) was called and in that logic, DoAction is only called on the first enemy party member. This means that in practice, whenever `firststrike` is true, actionid should ALWAYS be 0. The exception clause can be considered dead logic.

### Player action
This phase applies only if the entity has a `Player` tag.

#### Player action setup
The setup first begins by initialising a local variable called targetentity which is the actor data of the action's target (if it exists). This is primarily used for an [itemarea](../../Player%20UI/AttackArea.md) of `SingleAlly` or `SingleEnemy` targetting action, but it is technically initialised for multi targetting, just not very useful. Only some action uses this variable and it may not get a valid actor if it is not needed, but it must be set correctly if it is. Here's the logic for determining the value:

- If actionid is -555 (blank dummy call), the value is left at the [BattleData](../../Actors%20states/BattleData.md)'s default
- Otherwise, if [currentaction](../../Player%20UI/Pick.md) is `ItemList` (this is an item use action), then it depends on [itemarea](../../Player%20UI/AttackArea.md) (if none match, the variable is left at the [BattleData](../../Actors%20states/BattleData.md)'s default):
    - `SingleAlly`: The `target` player party member
    - `SingleEnemy`: [GetAvaliableTargets](../../Actors%20states/Targetting/GetAvaliableTargets.md) is called without onlyground and onlyfront with excludeunderground using actionid as the attack id followed by setting the variable to `avaliabletargets[target]`. NOTE: In practice, the GetAvaliableTargets shouldn't change anything under normal gameplay because it was called with similar parameters (attackid was -1, but it doesn't change anything) during [SetItem](../../Player%20UI/SetItem.md)
- Otherwise, if `target` is less than the length of `avaliabletargets` (it's a valid target), the variable is set to `avaliabletargets[target]`
- Otherwise, it's left at [BattleData](../../Actors%20states/BattleData.md)'s default

After, if actionid isn't negative (meaning this isn't a basic attack action or player first strike) and [currentaction](../../Player%20UI/Pick.md) isn't `ItemList` (meaning this must be a [skill](../../../Enums%20and%20IDs/Skills.md)), the cost of the skill is paid by doing the following:

- If the `HPFunnel` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped or [flagvar](../../../Flags%20arrays/flagvar.md) 0 is negative (meaning it's an HP cost), the `hp` of the `currentturn` player party member is decreased by the absolute value of flagvar 0
- Otherwise, instance.`tp` is decreased by [flagvar](../../../Flags%20arrays/flagvar.md) 0 (TP cost)

If a cost was paid, usedtp is set to the value of [flagvar](../../../Flags%20arrays/flagvar.md) 0 (the TP or HP cost).

After, if actionid isn't -555 (meaning it's not a player first strike) and [currentaction](../../Player%20UI/Pick.md) isn't `ItemList` (meaning it's not an item use so it's a basic attack or skill action), `checkingdead` is set to a new [UseCharm](../UseCharm.md) call with the type being `AttackUp`.

After, `targettedenemy` is set to the `enemydata` index of the targetted enemy using the following logic:

- If actionid is -555 (blank dummy call), it's 0
- Otherwise, if `target` is at least `avaliabletargets`'s length - 1 (this is wrong, but safe, see the note below) or [currentaction](../../Player%20UI/Pick.md) is `ItemList` (this is an item use action), it's `target`
- Otherwise, it's the battleentity.`battleid` of the return of GetEnemyFromAvaliable using `avaliabletargets[target]`. Basically, it means if `avaliabletargets[target]`'s battleentity still exists in `enemydata`, it will be its `battleid` (which is the same than the `enemydata` index) and if it doesn't, it will be 0

NOTE: This logic is very broken, but in practice, it's only ever used in 2 actions: the `HeavyStrike` [skill](../../../Enums%20and%20IDs/Skills.md) or `Beetle`'s basic attack. For those actions specifically, this logic happens to always be correct whether it is on accident or not. `targetedenemy` will be correct in these cases, but it should not be relied upon because its value isn't reliable and can easilly point to the wrong enemy party member.

Finally, all frames are yielded while `checkingdead` is in progress (which would be the [UseCharm](../UseCharm.md) call made earlier if it was).

#### Player action procedure
The actionid directly tells what action will be performed by a switch on it. Unlike enemy actions, there is no loop so the action is performed once and then completes.

TODO: document the player actions in part 3 of the battle docs in a separate place

### Enemy action
This phase applies if the entity doesn't have a `Player` tag unless it's not the first `enemydata` whenever `firststrike` is true, but as mentioned in the setup phase, this never happens under normal gameplay.

#### Enemy action setup
This part of the enemy action phase occurs before any action is performed:

- `checkingdead` is set to a new [UseCharm](../UseCharm.md) call with the type being `DefenseUp`
- All frames are yielded while `checkingdead` is in progress

The action can only occur if the enemy party member doesn't have a [Flipped](../../Actors%20states/BattleCondition/Flipped.md) condition for 2 turns or more (the action is allowed if there's 1 turn left on it or the condition isn't present). If it doesn't occur, the coroutine skips to the post action phase.

If the enemy party member has either the [Flipped](../../Actors%20states/BattleCondition/Flipped.md) or [Topple](../../Actors%20states/BattleCondition/Topple.md) conditions for exactly 1 turn left, the following occurs:

- Both the [Flipped](../../Actors%20states/BattleCondition/Flipped.md) and [Topple](../../Actors%20states/BattleCondition/Topple.md) conditions are removed via [RemoveCondition](../../Actors%20states/Conditions.md) on the enemy party member
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the enemy party member with a height of 10.0
- entity.`overrideanim` is set to false
- entity.`basestate` and entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- startstate is set to 0 (`Idle`)
- 0.1 seconds are yielded

After, a couple of local variables are initialised which are enemy actions specific:

- chance: A random integer between 0 and 100 exclusive (100 possible outcome)
- chances: A float array starting at null
- hardmode: A boolean that is true if any of the following is true (false if none are):
    - The `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped
    - [flags](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
    - [flags](../../../Flags%20arrays/flags.md) 166 is true (EX mode active on the B.O.S.S system)
- pos: A vector starting at Vector3.zero

After, if the enemy party member [isdefending](../../Actors%20states/Enemy%20features.md#isdefending):
    
- The enemy party member's `isdefending` is set to false
- startstate is set to entity.`basestate`
- entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to its `basestate`

Finally, all frames are yielded while entity is not `onground`.

#### Enemy action loop
Enemy actions works differently than player action in the sense they are all executed inside an infinite loop. This gives the ability for an action to perform many iterations before completing.

The action logic is determined by a giant switch on the enemy party member's `animid` which is its [enemy](../../../Enums%20and%20IDs/Enemies.md) id.

TODO: Document the enemy actions in part 3 of the battle docs in a separate place.

### Post action
This phase occurs after any actions. It starts with logic that allways occur no matter the action:

- If `demomode` is true, the `helpbox` is destroyed if it existed
- If entity has the `Enemy` tag, actionid is overriden to its corresponding enemy party member index
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called

From there, what follows depends on if the actionid is -555 or not (whether this is a blank dummy call or not).

If it is, the only logic that happens is `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md) followed by skipping to the cleanup phase.

If it's not, the full post action logic occurs with 2 distinct branche:

- fled is false meaning the action didn't signaled that the enemy party member fled the battle
- fled is true meaning the action signaled that the enemy party member fled the battle

In either cases, they both end by setting all enemy party members's `lockposition` to false once they are done.

#### No fled enemy post action

- entity.`overrideanimspeed` is set to false
- entity.`spin` is zeroed out
- `playertargetID` is reset to -1
- `killinput` is set to true
- FlipAngle is called on the entity
- entity.`overrideanim` is reset to false
- entity.`overrideflip` is set to true
- entity.`rigid` gravity is enabled
- 0.15 seconds are yielded
- If the distance with the entity's position and startp is greater than 0.35, PlayMoveSound is called with the entity which plays its corresponding move sound if one exists for its [animid](../../../Enums%20and%20IDs/AnimIDs.md) using the audio source 9
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move towards startp with a multiplier of 2.0 using its `walkstate` as the state and startstate as the stopstate
- All frames are yielded while the entity is in a `forcemove`
- The audio source 9 has its sound stopped if applicable (this is the move sound played earlier if one existed)
- A frame is yielded
- entity's position is set to startp
- entity.`flip` is reset to flip
- entity.`overrideflip` is reset to false
- 0.15 seconds are yielded
- If randomposafter is true, an attempt with be made to change the enemy party member's [position](../../Actors%20states/BattlePosition.md) by generating a 50/50 random position between `Ground` and `Flying`. If the position changed from its previous one, the following is performed (in either cases, entity.`oldid` is reset to -1 after):
    - If the new enemy party member's [position](../../Actors%20states/BattlePosition.md) is `Ground`:
        - As long as entity.`height` is higher than entity.`minheight`, the `Fall` AnimationClip is played on entity.`anim` followed by entity.`height` being decreased by 0.075 of the game's frametime followed by a frame yield
        - entity.`onground` is set to true
        - entity.`height` is set to entity.`minheight`
        - entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to its `basestate`
        - A frame is yielded
        - If the entity.`originalid` isn't the `BeeBot` [animids](../../../Enums%20and%20IDs/AnimIDs.md), entity.`basestate` in enum string format is played on entity.`anim` (this implies entity.`basestate` must be a [predefind animation](../../../Entities/EntityControl/Animations/animstate.md#predefined-animations-names))
    - Otherwise (the new [position](../../Actors%20states/BattlePosition.md) is `Flying`):
        - `Idlef` is played on entity.`anim`
        - All frames are yielded while entity.`height` is less than 1.9 (entity.`height` is increased by 0.075 of the game's frametime each time before the frame yield)
        - entity.`bobrange` is reset to entity.`startbf`
        - entity.`bobspeed` is reset to entity.`startbs`
        - entity.`height` is set to 2.0
    - [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md#updateanimspecific) is called on the entity
- If the entity has the `Player` tag (this is a player party member):
    - If usedtp is above 0, nocharm is false and [currentaction](../../Player%20UI/Pick.md) isn't `ItemList` (meaning a skill was used that had a paid cost to it while healing charms aren't disallowed):
        - [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to usedtp / 2 clamped from 1 to usedtp (the integer division floors implicitly)
        - `checkingdead` is set to a new [UseCharm](../UseCharm.md) call with the type being `HealTP` which heals for the amoutn set to flagvar 1 just before
        - All frames are yielded while `checkingdead` is in progress
    - `playerdata[currentturn]`'s `tired` is incremented if [currentaction](../../Player%20UI/Pick.md) isn't `ItemList` and actionid is not among the following (these are all the team moves, the reason they are exempted from exhaustion is because their individual action logic already took care of incremented the proper `tired` fields so this would have been a second, unwanted increment):
        - 5 (`BeeFly`)
        - 26 (`IceBeemerang`)
        - 27 (`IceDrill`)
        - 31 (`IceSphere`)
    - `playerdata[currentturn]`'s `tired` is incremented again if `turns` is 0 (meaning this is the first turn of the battle) while the `StrongStart` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped
    - Unless `dontusecharge` is true, `playerdata[currentturn]`'s `charge` is reset to 0
    - [EndPlayerTurn](../EndPlayerTurn.md) is called
    - [currentaction](../../Player%20UI/Pick.md) is set to `BaseAction`
    - `option` is set to `lastoption` (this restores the main vine menu's selection to whichever was the last one selected)
    - If [GetFreePlayerAmmount](../../Actors%20states/Player%20party%20members/GetFreePlayerAmmount.md) returns at least 1, [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- Otherwise, if `selfsacrifice` is false (the enemy party member didn't killed themselves during their action):
    - If the enemy party member had any `delayedcondition`:
        - All `delayedcondition` are processed. See the [delayed condition](../../Actors%20states/Delayed%20condition.md) documentation to learn more
        - If any `delayedcondition`'s processing caused a [condition](../../Actors%20states/Conditions.md) to be inflicted, the enemy party member's `cantmove` is set to 0 (EndEnemyTurn will later advance it making it 1 which means one actor turn needs to pass before an action is available)
        - If the enemy party member's [position](../../Actors%20states/BattlePosition.md) is `Flying` while its [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) is false, entity.`droproutine` is set to a new [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) coroutine on the entity
        - 0.5 seconds are yielded
        - The enemy party member's `delayedcondition` is set to null
    - If nocharm is false (healing charms weren't disallowed):
        - `checkingdead` is set to a new [UseCharm](../UseCharm.md) call with the type being `HealHP`
        - All frames are yielded while `checkingdead` is in progress
    - If the enemy party member was performaning a [hitaction](../../Actors%20states/Enemy%20features.md#hitaction), `enemy` is set to false, giving control back to the player party
    - [EndEnemyTurn](../EndEnemyTurn.md) is called with the actionid
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- If this isn't a `firststrike`:
    - A [CheckDead](CheckDead.md) is done without storing the coroutine
    - Otherwise, `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- If this isn't a `firststrike` and any enemy party member has their battleentity.`droproutine` in progress:
    - `action` is set to true switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
    - `startdrop` is set to true which allows any existing enemy party members to fall during their `droproutine`
    - All frames are yielded while an enemy party member still has its battleentity.`droproutine` in progress (it also constantly set `startdrop` to true to make sure they can drop)
    - `startdrop` is reset to false
    - `action` is reset to false switching back to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)

#### Fled enemy post action

- [EndEnemyTurn](../EndEnemyTurn.md) is called witht the actionid
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- A [CheckDead](CheckDead.md) is performed, but the coroutine isn't stored anywhere.

### Cleanup
This is the last phase of the coroutine:

- `selfsacrifice` is reset to false
- A frame is yielded
- `lastkill` is reset to -1
