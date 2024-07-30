# `Pisci`

## Assumptions
It is assumed this enemy is fought alongside [Cenn](Cenn.md) as they have several interactions together that would work unexpectedly if a fight is started without the presence of both enemy party members at once.

It is also assumed that this enemy is loaded with their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to the [proper EventDialogue](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md) because it allows the first fight to end early in case this enemy dies before their [HPPercent](../../Actors%20states/HPPercent.md) becomes lower than 0.5. On the second fight, the [StartBattle](../../StartBattle.md) logic will disable it.

## [StartBattle](../../StartBattle.md) special logic
This enemy has special logic after being loaded in StartBattle:

- If [flags](../../../Flags%20arrays/flags.md) 497 is true (after beating [Cenn](Cenn.md) and `Pisci` the first time):
    - `eventondeath` is set to -1 which disables their [EventDialogue](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md)
- Otherwise (the player is fighting them for the first time):
    - [AlwaysSurvive](../../Damage%20pipeline/AttackProperty.md) is added to this enemy's [weakness](../../Actors%20states/Enemy%20features.md#weakness)
- `battlepos` is increased by 1.0 in x
- position is set to the new `battlepos`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: A cooldown for using the `PisciWall` summon moves. When the move is used, the value is set to 2 and it gets decremented in the pre move logic if it is higher than 0. The summon move only becomes usable again if the value is 0 by the time the move selection process occurs and the enemy isn't present already. Effectively, it's an antispam of 1 actor turn on the move's usage

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to heal [Cenn](Cenn.md) by 4 `hp` when their [HPPercen](../../Actors%20states/HPPercent.md) is lower than 0.75 in the pre move logic changes to 41% from 31%
- In the rocks projectiles throw move, the amount of hits to do changes to be random between 2 and 3 instead of being random between 2 and 4
- In the rocks projectiles throw move, the speed of each Projectile calls changes to be random between 38.0 and 44.0 instead of being always 47.5

## Move selection
2 moves are possible:

1. A multiple targets rocks projectiles throw
2. Summons a `PisciWall` enemy

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/2|
|2|1/2|

However, if move 2 is selected and a `PisciWall` is already present or that `data[0]` is higher than 0 (meaning the cooldown on it hasn't expired yet), move 1 will be used instead.

## Pre move logic
The following logic always happen before using a move. It's in 4 parts done in order: `data[0]` decrement, the first fight event logic, the infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition logic and the healing [Cenn](Cenn.md) logic.

### `data[0]` decrement
This part only happens if `data[0]` is higher than 0 (the `PisciWall` summon cooldown hasn't expired yet):

- `data[0]` gets decremented which is the actor turn cooldown on summoning a `PisciWall` enemy

### First fight event logic
This part only happens if [flags](../../../Flags%20arrays/flags.md) 497 is false meaning this is the first time to player fights [Cenn](Cenn.md) and `Pisci`:

- If [HPPercent](../../Actors%20states/HPPercent.md) of this enemy or [Cenn](Cenn.md) is lower than 0.5 (if they are present):
    - [EventDialogue 15](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md#cenn-and-pisci-eventdialogue) is triggered
- Yield all frames until `inevent` is false (if the EventDialogue above was triggered, it effectively stalls DoAction entirely until the battle almost ends)

> NOTE: If the EventDialogue is triggered here, it won't prevent DoAction from running while [ReturnToOverworld](../../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) without flee is ongoing. This creates a problem because the EventDialogue runs a [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) which clears `enemeydata` due to killing [Cenn](Cenn.md) and `Pisci` and all this happens BEFORE DoAction gets to resume. Since DoAction can't operate on an empty `enemydata`, it will immediately throw an IndexOutOfRange excpetion. Fortunately, this throw is inconsequential because [ReturnToOverworld](../../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) is still ongoing and the battle would already be in a [terminal flow](../../Battle%20flow/Update%20flows/Terminal%20flow.md) by this point. Effectively, the battle will still end without consequences because DoAction can't race against this process anyway before it throws and gets destroyed due to the battle teardown process.

### Infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition logic
This part only happens if [Cenn](Cenn.md) is not present in the enemy party and that this enemy doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition:

- `Wam` sound plays
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote for 30 time
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 999 main turns (effectively infinite) with effect
- Yield for 0.5 seconds

### Healing [Cenn](Cenn.md) logic
This part happens if all of the following conditions are fufilled:

- `Cenn` is present
- `Cenn`'s [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.75
- A 31% RNG check passes (41% instead if hardmode is true)

If all of the above are fufilled:

- `Ping2` sound plays
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote for 30 time
- Yield for 0.5 seconds
- A new sprite object is created childed to the `battlemap` using the `CrunchyLeaf` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy position + its `cursoroffset` + (-0.4, -0.1, -0.1)
- animstate set to 4 (`Angry`)
- `ItemHold` sound plays
- Yield for a second
- The item sprite gets destroyed
- animstate set to the local startstate
- [Heal](../../Actors%20states/Heal.md) called to heal `Cenn` by 4 `hp`
- Yield for 0.5 seconds

## Move 1 - Rocks projectiles throw
A multiple targets rocks projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 to 4 times (2 to 3 times instead if hardmode is true), but the call and further calls won't happen if there's not at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|null|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new sprite object rooted using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite positioned at this enemy position + (-0.5, 1.5, -0.1) with a scale of 0.85x|47.5 (random between 38.0 and 44.0 instead if hardmode is true)|3.5|null|null|null|null|(0.0, 0.0, 25.0)|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called (but this call won't do anything)
- Camera moves to look near the midpoint between this enemy and `playerdata[playertargetID]`
- The amount of hits is determined. It's random between 2 and 4 inclusive (between 2 and 3 inclusive instead if hardmode is true). A SpriteRenderer array is initialised with this amount as the length to hold the projectiles objects
- For each hit to do, if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - animstate set to 100
    - `Toss13` sound plays
    - Yield for 0.166 seconds
    - The projectile element is set to a new sprite object rooted using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite positioned at this enemy position + (-0.5, 1.5, -0.1) with a scale of 0.85x
    - Projectile 1 call happens
    - Yield for a random interval between 0.45 and 0.65 seconds
    - animstate set to 0 (`Idle`)
    - Yield for a frame
- Yield all frames until all projectiles objects are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 2 - `PisciWall` summon
Summons a `PisciWall` enemy. No damages are dealt.

The `PisciWall` is an enemy that has no action logic and thus doesn't need a documentation page. Its only purpose is to be hinderance since it is always the first enemy party member whose [poisition](../../Actors%20states/BattlePosition.md) is `Ground` which affects some player actions's targetting.

### Logic sequence

- `data[2]` is set to 2 (prevents the usage of this move on the next actor turn if `PisciWall` isn't present by that time)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-0.15, 0.0, 0.0) with 2.0 multiplier using 1 (`Walk`) as walkstate and 101 as stopstate
- Camera moves to look near `forcetarget`
- Yield all frames until `forcemove` is done
- `Dig2` sound plays with 0.7 pitch
- `ContinuousSmokeSphere` particles plays at this enemy position with -1 alivetime and 0.5x scale
- Yield for 0.5 seconds
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a `PisciWall` with type `FromGroundKeepScale` at (-0.25, 0.0, -0.1)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- `enemydata[lastaddedid]`'s [diebyitself](../../Actors%20states/Enemy%20features.md#diebyitself) is set to true which allows [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md) to kill the summoned enemy if they are the only one remaining in the enemy party
- The `ContinuousSmokeSphere` particles gets moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 0.5 seconds
