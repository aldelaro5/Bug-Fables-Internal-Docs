# `Cenn`

## Assumptions
It is assumed this enemy is fought alongside [Pisci](Pisci.md) as they have several interactions together that would work unexpectedly if a fight is started without the presence of both enemy party members at once.

It is also assumed that this enemy is loaded with their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to the [proper EventDialogue](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md) because it allows the first fight to end early in case this enemy dies before their [HPPercent](../../Actors%20states/HPPercent.md) becomes lower than 0.5. On the second fight, the [StartBattle](../../StartBattle.md) logic will disable it.

## [StartBattle](../../StartBattle.md) special logic
This enemy has special logic after being loaded in StartBattle:

- If [flags](../../../Flags%20arrays/flags.md) 497 is true (after beating `Cenn` and [Pisci](Pisci.md) the first time):
    - `eventondeath` is set to -1 which disables their [EventDialogue](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md)
- Otherwise (the player is fighting them for the first time):
    - [AlwaysSurvive](../../Damage%20pipeline/AttackProperty.md) is added to this enemy's [weakness](../../Actors%20states/Enemy%20features.md#weakness)
- `battlepos` is increased by 1.0 in x
- position is set to the new `battlepos`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to heal [Pisci](Pisci.md) by 4 `hp` when their [HPPercen](../../Actors%20states/HPPercent.md) is lower than 0.75 in the pre move logic changes to 41% from 31%
- In the stick boomerang throw move, the amount of frames the stick takes to reach its target changes to 36.0 on the first movement and 46.0 frames on the second movement instead of being 44.0 frames on the first movement and 54.0 frames on the second movement

## Move selection
2 moves are possible:

1. A single target double hitting stick boomerang throw
2. A single target stick slash attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/2|
|2|1/2|

## Pre move logic
The following logic always happen before using a move. It's in 3 parts done in order: the first fight event logic, the infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition logic and the healing [Pisci](Pisci.md) logic.

### First fight event logic
This part only happens if [flags](../../../Flags%20arrays/flags.md) 497 is false meaning this is the first time to player fights `Cenn` and [Pisci](Pisci.md):

- If [HPPercent](../../Actors%20states/HPPercent.md) of this enemy or [Pisci](Pisci.md) is lower than 0.5 (if they are present):
    - [EventDialogue 15](../../Battle%20flow/EventDialogues/Cenn%20and%20Pisci.md#cenn-and-pisci-eventdialogue) is triggered
- Yield all frames until `inevent` is false (if the EventDialogue above was triggered, it effectively stalls DoAction entirely until the battle almost ends)

> NOTE: If the EventDialogue is triggered here, it won't prevent DoAction from running while [ReturnToOverworld](../../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) without flee is ongoing. This creates a problem because the EventDialogue runs a [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) which clears `enemeydata` due to killing `Cenn` and [Pisci](Pisci.md) and all this happens BEFORE DoAction gets to resume. Since DoAction can't operate on an empty `enemydata`, it will immediately throw an IndexOutOfRange excpetion. Fortunately, this throw is inconsequential because [ReturnToOverworld](../../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) is still ongoing and the battle would already be in a [terminal flow](../../Battle%20flow/Update%20flows/Terminal%20flow.md) by this point. Effectively, the battle will still end without consequences because DoAction can't race against this process anyway before it throws and gets destroyed due to the battle teardown process.

### Infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition logic
This part only happens if [Pisci](Pisci.md) is not present in the enemy party and that this enemy doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition:

- `Wam` sound plays
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote for 30 time
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 999 main turns (effectively infinite) with effect
- Yield for 0.5 seconds

### Healing [Pisci](Pisci.md) logic
This part happens if all of the following conditions are fufilled:

- `Pisci` is present
- `Pisci`'s [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.75
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
- [Heal](../../Actors%20states/Heal.md) called to heal `Pisci` by 4 `hp`
- Yield for 0.5 seconds

## Move 1 - Stick boomerang throw
A single target double hitting stick boomerang throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|
|2|Always happen|This enemy|Same target as DoDamage 1|1|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playerdata[playertargetID]`
- animstate set to 102
- Yield for 0.5 seconds
- `Woosh4` sound plays
- animstate set to 103
- `Woosh` sound plays on loop using `sounds[9]` with 1.2 pitch
- A new sprite object is created childed to the `battlemap` using the `projectilepsrites[16]` sprite (a branch stick) positioned at this enemy position + (-0.35, 1.25, -0.1)
- Over the course of 44.0 frames (36.0 frames instead if hardmode is true), the stick moves to a SmoothLerp result via a lerp to its position where it changes to `playertargetentity` position + (-7.5, 1.25, -0.1) where the first half is a lerp and the second half is a SmoothStep of each of the vector's components. Before each frame yield, the stick's z angle is increased by 25.0x the game's frametime. On th first frame that the stick x position becomes lower than `playertargetentity` x position, DoDamage 1 call happens
- Over the course of 54.0 frames (46.0 frames instead if hardmode is true), the stick moves to a SmoothLerp result via a lerp to its position where it changes to this enemy position + (0.9, 1.25, -0.1) where the first half is a lerp and the second half is a SmoothStep of each of the vector's components. Before each frame yield, the stick's z angle is increased by 25.0x the game's frametime. On th first frame that the stick x position becomes higher than `playertargetentity` x position, DoDamage 2 call happens
- `sounds[9]` is stopped
- The stick object is destroyed
- animstate set to 104
- Yield for 0.5 seconds

## Move 2 - Stick slash attack
A single target stick slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- Yield all frames until `forcemove` is done
- `Slash2` sound plays
- Yield for 0.5 seconds
- animstate set to 101
- DoDamage 1 call happens
- Yield for 0.5 seconds
