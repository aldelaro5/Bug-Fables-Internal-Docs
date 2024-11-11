# AdvanceMainTurn
This is an action coroutine that runs at the very end of every main turn when both parties no longer can act for this turn.

## Procedure

- `commandsuccess` is reset to false
- `blockcooldown` is reset to 0.0
- `action` is set to true which changes to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- If the following conditions are met, [delprojs](../../Actors%20states/Delayed%20projectile.md) are processed (see the section below for details):
    - `delprojs` isn't empty
    - There are at least one player party member who has their `hp` above 0 while not `eatenby`
    - There are at least one enemy party member who has their `hp` above 0
    - `gameover` isn't in progress
- `nonphyscal` is set to false
- Each player party member gets their actor turn advanced (see the section below for details)
- A frame is yielded
- `availableplayers` is set to the amount of player party members whose `hp` is above 0
- If `availableplayers` is 0, [DeadParty](../Terminal%20coroutines/DeadParty.md) is started followed by an abrupt yield break. This will change to a [terminal flow](../Update%20flows/Terminal%20flow.md) as it is a terminal coroutine.
- If there is at least one enemy party member, the enemy party section of the turn advancement is performed alongside some end of main turn process (see the section below for details)
- Otheriwse if `gameover` was already in progress, a yield break is issued because it means we already changed to a [terminal flow](../Update%20flows/Terminal%20flow.md) and nothing is left to do
- Otherwise: 
    - `cancelupdate` is set to true changing to a [terminal flow](../Update%20flows/Terminal%20flow.md)
    - A second is yielded
    - If `inevent` is false (we weren't in an [EventDialogue](../EventDialogue.md)), [AddExperience](../Terminal%20coroutines/AddExperience.md) is started changing to a [terminal flow](../Update%20flows/Terminal%20flow.md). NOTE: the coroutine still continues, but in practice, it can't race against AddExperience
- If `actedthisturn` is false, `noaction` is incremented
- If `gameover` isn't in progress while `noaction` is 5 (meaning 5 main turns passed without the ability for the player party to act), the inaction failsafe is done (see the section below for more details)
- `actedthisturn` is set to false
- `forceattack` is set to -1
- `playertargetID` is set to -1
- If `damagethisturn` is higher than [flagvar](../../../Flags%20arrays/flagvar.md) 41 (highest damage in one turn), the flagvar value is set to `damagethisturn`
- `damagethisturn` is set to 0
- RefreshAllData is called which sets `alldata` to a new list which consists of all the `playerdata` followed by all the `enemydata` appended together
- [UpdateEntities](../../Visual%20rendering/UpdateEntities.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- `option` is set to `lastaction`
- If there is at least one enemy party member, [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is invoked again in 0.75 seconds
- If `charmcooldown` is above 0, it is decremented
- `chompyattacked` is set to false
- `hideenemyhp` is set to false
- `mainturn` is set to null (which signals the coroutine is no longer in progress as no yield will be done from now on)
- `blockcooldown` is set to 0.0 (this resets the [GetBlock](../GetBlock.md) counter so blocking becomes inactive)
- All GameObjects with tag `DestroyTurn` have their tag set to `Untagged` followed by their destruction in 5.0 seconds

### `delprojs` advance
`delprojs` (or [delayed projectiles](../../Actors%20states/Delayed%20projectile.md#delayedprojectiledata)) are projectiles that an enemy can launch and the projectile will only hit the targetted player party member after a certain amount of main turns passed. This section processes them.

The first thing that happens is `nonphyscal` is set to true (it will be set to false after this section).

From there, for each `delprojs` (going from last to start), the delproj's `turns` gets decremented and if it reached 0, it will land by doing the following:

- instance.`camtarget` is set to null
- instance.`camtargetpos` is set to (-4.25, 0.0, 0.0)
- instance.`camoffset` is set to (0.0, 4.0, -8.5)
- `enemy` is set to true
- `blockcooldown` is set to 0.0
- `commandsuccess` is set to false
- If the delproj has a `whilesound` set, it is played on loop unless the first character is `@` in which case, it will play the sound only once where name is the one without the `@`
- The `args` of the delproj are parsed as explained in the [delayed projectile documentation](../../Actors%20states/Delayed%20projectile.md#args-field)
- Unless a `noshadow` command was parsed, a ShadowLite is added to the delproj's `obj`
- For delproj's `framestep` amount of frames, the projectile is moved (kept track with a local frame counter starting at 0.0 and getting incremented by MainManager.`framestep` each iteraction):
    - The delproj's `obj` psotion is set to a lerp from the initial one to the `partypos` of the delproj's `position` + the `move` command offset if it was parsed with a factor of the ratio between the amount of frames spent so far on this loop vs delproj's `framestep`
    - A frame is yielded
- If the delproj had a `whilesound`, it is stopped. NOTE: if this wasn't a looped sound, the call won't work, but the sound will stop on its own
- If the delproj has a `deathsound`, it is played
- If the delproj has `deathparticle`, they are played at the `obj` position + the `partoff` command offset if it was parsed
- The delproj's `obj` is destroyed
- If the player party member whose index is the `partypointer` corresponding to the delproj's `position` has an `hp` above 0 and doesn't have an `eatenby`, some logic is performed on this player party member:
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called targetting the player party memmber without an attacker for delproj's `damage` damageammount with delproj's `property` as the property and `commandsuccess` as the block
    - If the `hp` of the player party member reached 0 or below, their `turnssincedeath` is set to -1
- [RemoveDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#removedelayedprojectile) is called on the delproj which removes it from `delprojs`
- `enemy` is set to false
- 0.6 seconds are yielded

If any `delprojs` landed:

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- `checkingdead` is set to a [CheckDead](CheckDead.md) action coroutine starting
- All frames are yielded while `checkingdead` is in progress
- `action` is set again to true
- A frame is yielded
- All actors from both parties with a `cantmove` below 1 (at least one action available) gets their `cantmove` clamped from 1 to 99 (forcing at least one turn needed before an action is available). This in practice doesn't do anything destructive because `moreturnnextturn` is granted later correctly and it's not possible to have a meaningul `cantmove` above 0 by this point (if it happened, it likely means the player party member [IsStopped](../../Actors%20states/IsStopped.md) which imply that when the stopping [condition](../../Actors%20states/Conditions.md) gets removed, the `cantmove` will reset anyway).
- `currentturn` is set to 1 (deselects any previously selected player party member)
- 0.5 seconds are yielded

### Player party advance
This section happens for each player party member.

- The corresponding `receivedrelay` of the player party member index is set to false
- [AdvanceTurnEntity](../AdvanceTurnEntity.md) is called on the player party member with hasdelay starting at false
- If the turn advancement resulted in hasdelay becoming true, 0.75 seconds are yielded
- If `eatenkill` is true (meaning a player party member died as a result of an eating attack) the game needs to properly kill the player party member that died by doing the following:
    - `eatenkill` is set to false
    - All frames are yielded while [spitout](../../Actors%20states/BattleCondition/Eaten.md#spitout) is in progress
    - `checkingdead` is set to a new [CheckDead](CheckDead.md) coroutine starting
    - All frames are yielded while `checkingdead` is in progress
- battleentity.`shakeice` is set to false

Finally, if the `MiracleMatter` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped on the player party member, a revival process will occur, but it can only happen if on top of the medal being equipped, all the following are true:

- This isn't a [firststrike](../firststrike%20system.md) from the enemy party
- At least one player party member has its `hp` above 0 without being `eatenby`
- `lockmmater` is false
- The player party member has its `turnssincedeath` be at least (2 - amount of `MiracleMatter` equipped - 1) after clamping from 1 to 3 (essentially, at least 2 if only one is equipped, at least 1 if 2, at least 0 if more is equipped, but this case isn't possible under normal gameplay)

If all the conditions are met, the revive occurs as follows:

- `turnssincedeath` is set to 0
- [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called on the player party member index with 2 hp and with showcounter
- 0.5 seconds are yielded
- If the player party member's `lockcantmove` is false, their `cantmove` is set to 0 (this gives them one action available)
- The player party member's `lockcantmove` is set to false

### Enemy party advance and end of main turn process
This section not only manages enemy turn advance, but it also does some end of main turn process because being in this section means the battle is not going to end.

The first thing that happens is relating to the `FavoriteOne` [medal](../../../Enums%20and%20IDs/Medal.md) which is tracked by `attackedally`. If it's not negative, it means the player party member with the matching `trueid` was attacked and the other members should receive the medal's benefits. This is only done if there is more than 1 player party member's `hp` above 0 while not being `eatenby`.

NOTE: This imply that if the `attackedally` player party member got hit as well as another player party member on the same main turn, the logic won't happen because the game requires more than 1 player party members alive. This isn't correct: it should require at least 1 player party member alive instead of 2 or more.

The process to apply the medal is as follows:

- For each player party member whose `trueid` isn't `attackedally` with an `hp` above 0 and without the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition:
    - battleentity.`emoticoncooldown` is set to 40.0
    - `didnothing` is set to false
    - battleentity.`emoticonid` is set to 2 (red ! mark)
- If at least one player party member had an emote set, the `Wam` sound is played
- All frames are yielded while the last player party member's battleentity.`emoticoncooldown` is above 0.0
- For each player party member whose `trueid` isn't `attackedally` with an `hp` above 0 with a `charge` less than 3 and without the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition:
    - A [StatEffect](../../Visual%20rendering/StatEffect.md) of type 4 (green up arrow) coroutine is started on the player party member
    - `charge` is incremented
- If at least one player party member gained a `charge`, the `StatUp` sound is played at 1.25 pitch
- 0.5 seconds are yielded
- `attackedally` is reset to -1

From there, each enemy party members gets their turn advanced by having the following logic apply to each of them:

- [AdvanceTurnEntity](../AdvanceTurnEntity.md) is called on the eneny party member with hasdelay starting at false which advances their actor turn
- If hasdelay got a value of true after the turn advancement, 0.75 seconds are yielded
- battleentity.`shakeice` is set to false
- If `cantmove` is 0 (they would have only 1 action available after the actor turn), `cantmove` is set to -`moves` + 1 (this resets the available actions counter to the base one set from the normal amount being [moves](../../Actors%20states/Enemy%20features.md#moves))
- battleentity.`spin` is zeroed out
- `turnsnodamage` is incremented (this value is only written to, but never read so it's UNUSED)

Finally, some end of main turn process logic occurs:

- SetLastTurns is called which resets `lastturns` elements to be all -1 which resets the player party member selection cycle
- A frame is yielded
- `turns` is incremented
- `checkingdead` is set to a EndOfTurnMedals coroutine starting followed by all frames being yielded while the coroutine is in progress. This is what the coroutine does:
    - Every 2 `turns`, for each player party member with the `HappyHeart` [medal](../../../Enums%20and%20IDs/Medal.md) equipped and whose `hp` is above 0:
        - [Heal](../../Actors%20states/Heal.md) is called on the player party member with the amount being 2 * the amount of `HappyHeart` medals equipped
        - 0.5 seconds are yielded
    - Every 2 `turns` (same cycle as the one above), if the `HappyTP` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped:
        - HealTP is called which plays a `Heal2` sound followed by a heal of instance.`tp` by 2 * the amount of `HappyTP` medals equipped clamped from 0 to instance.`maxtp` followed by a [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) with type 2 (TP), the amount being the amount of tp healed, the start position being `playerdata[`[GetRandomAvaliablePlayer()](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md)`].battleentity` position + (2.0, 2.0, 2.0) and the end position being (5.0, 5.0, 5.0)
        - 0.5 seconds are yielded
    - `checkingdead` is set to null which informs the caller the coroutine has ended so it can stop yielding
- `option` is set to 0
- `currentturn` is set to -1 (this unselects any players party members previously selected)
- `enemy` is set to false (this resets the [controlled flow](../Update%20flows/Controlled%20flow.md) to be in the [player phase](../Main%20turn%20life%20cycle.md#player-phase))
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- `action` is set to false changing to a [controlled flow](../Update%20flows/Controlled%20flow.md#controlled-flow)
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

### Inaction failsafe
There is a failsafe in the game where if after 5 main turns, [PlayerTurn](../PlayerTurn.md) was never called meaning the player party could in no way perform any actions, the game will forcefully allow the player party to act again. This is tracked by `actedthisturn` being false reporting no action could be taken and the total amount of main turn is tracked by `noaction`. This failsafe exists presumably to mitigate a situation where the player party is stopped for too long notably through the results of stopping [conditions](../../Actors%20states/Conditions.md).

The failsafe is applied by doing the following:

- `noaction` is reset to 0
- For each player party member that [IsStopped](../../Actors%20states/IsStopped.md) without skipimmobile while their `hp` is above 0:
    - `MagicUp` particles are played at the battleentity position
    - [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on the player party member
    - Their `cantmove` is set to 0 making them able to act again
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- If at least one player party member had ClearStatus called on them earlier, the `Heal3` sound is played followed by a 0.5 seconds yield
