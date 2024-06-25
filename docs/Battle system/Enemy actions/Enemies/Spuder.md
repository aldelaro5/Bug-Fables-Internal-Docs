# `Spuder`

> NOTE: There are 5 related [EventDialogues](../../Battle%20flow/EventDialogues/Spuder.md) to this enemy action. They supplement the logic presented here in significant ways since it is expected that there are at least 3 different encounters with this enemy. Check the documentation to learn more about the first 2 encounter's changes to the action logic and for the 2 EventDialogues used on the third encounter. This page mostly talks about the logic on the third encounter which is the typical one and won't talk much about the first 2.

## [StartBattle](../../StartBattle.md#startbattle) special logic
If [flags](../../Flags%20arrays/flags.md) 41 is false (before the end of chapter 1) and either `flags` 613 (RUIGEE) or 614 (HARDEST) is true, the `hp` and `maxhp` are decreased by 15. This happens AFTER [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) was called to load this enemy and it happens regardless of the `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) being equipped or not.

## Relation with `SpuderReal`
`SpuderReal` is an enemy entry that has no action logic and it acts as a variant of `Spuder` meaning it ends up being loaded as a `Spuder` with the exception of some fields. It is only used when fighting them using the B.O.S.S system.

In practice, everything ends up being equivalent with one exception: `SpuderReal` has its [notired](../../Actors%20states/Enemy%20features.md#notired) set to true which means specifically for its B.O.S.S encounter, their exhaustion is disabled even when the `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is unequipped.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: Represents a main turn countdown that represents the state of the [position](../../Actors%20states/BattlePosition.md) of the enemy (it always starts with a value of 0):
    - -1: A temporary state where the enemy was `Flying` since 2 main turns and it decided to drop down to `Ground`, but all of the actor turns of the enemy aren't over yet. Once all of their actor turns are over, the value changes to 0
    - 0: The initial state, the enemy could decide to rise in the air to `Flying` on its own which will set the value to 2
    - 1: This enemy rose in the air to be `Flying` 2 main turns ago and it will drop down to `Ground` on the next main turn if it is still `Flying` by then which will change the value to -1
    - 2: This enemy rose in the air to be `Flying` on the last main turn and it will continue to be `Flying` on the next main turn if it is still `Flying` by then, but the value will decrease to 1

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true has 2 changes:

- In the poison breath move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the poison bubble throw move, each bubbles takes 1 / 0.02 frames (50.0) for the projectile to move to its target instead of 1 / 0.0175 frames (~57.1429)

## Move selection
4 moves are possible:

1. A Single target bite attack
2. Rise in the air
3. A party wide poison breath attack
4. A Single target poison bubble throw
5. Drop down from the air

Move 4 requires the enemy [position](../../Actors%20states/BattlePosition.md) `Flying`, but it is always selected if it is `Flying`. This logic applies to all of the 3 encounters.

Move 5 is always used at the start of the their first action of the main turn if `data[0]` is 1. However, it is the only move that won't end the action as `Spuder` will use move 1 or 3 in the same action right after. Notably, it sets `data[0]` to -1 which has effects on the move selection detailed further below.

As for moves 1 to 3 when on `Ground` or just dropped from `Flying`, their odds depends on whether or not its `hp` is less than their `maxhp` / 2 floored and whether `data[0]` is -1 or not. The latter is because the logic is built in such a way that `Spuder` cannot rise in the air to `Flying` if they already decided to drop down to `Ground` on this main turn. When that happens, it redirects the move selection to another move between 1 and 3 with different odds than normal.

Here is a table representing the different odds of using move 1, 2 or 3 depending on these factors:

|`hp` < `maxhp` / 2 floored?|`data[0]` is -1? (decided to drop down)|Move selection chances|
|--------------------------|---------------|----------------------|
|No|No|<ol><li>1/2</li><li>1/2</li><li>Never uses</li></ol>|
|No|Yes|<ol><li>4/5</li><li>Never uses</li><li>1/5</li></ol>|
|Yes|No|<ol><li>1/3</li><li>1/3</li><li>1/3</li></ol>|
|Yes|Yes|<ol><li>8/15</li><li>Never uses</li><li>7/15</li></ol>|

## Move 1 - Bite attack
A single target bite attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Scuttle2` sound plays via PlayMoveSound on loop using `sounds[9]` with 0.9 pitch and 0.8 volume
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) the `playertargetID` player party member postion + (1.1, 0.0, -0.25) with a multiplier of 1.3333334
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- animstate set to 105
- Yield for 0.2 seconds
- animstate set to 103
- `Bite` sound plays
- Yield for 0.3 seconds
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 2 - Rise in the air
Rise in the air to `Flying`. No damages are dealt, only the [position](../../Actors%20states/BattlePosition.md) changes to `Flying`.

This move will not be used if move 5 (drop down from the air) was used earlier in the same main turn.

### Logic sequence

- animstate set to 106
- New GameObject created called `line` with a LineRenderer childed to the [model](../../../Entities/EntityControl/Notable%20methods/AddModel.md)'s first child with several properties set such as a pure white color and the `spritemat` material
- `Chew` sound plays
- `line`'s second position is lerped towards the local startp + 15.0 in y over the course of 50.0 frames
- Yield for a frame
- `basestate`, `animstate` and the local startstate set to 108
- `Rise` sound plays
- `height` increases each frame for 0.25 of the game's frametime until it reaches 3.0 or above
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`
- `data[0]` set to 2
- Yield for 0.3 seconds
- If `tempdata` was 1 (meaning this was the second encounter), it is set to 0 so it won't override move selection again the same way. See the [second encounter](../../Battle%20flow/EventDialogues/Spuder.md#second-encounter) documentation to learn more

## Move 3 - Poison breath
A party wide poison breath attack

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any player party member whose `hp` is above 0.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|Each player party member whose `hp` is above 0 (dealt each in their own DoDamage call)|1|[poison](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`<sup>1</sup>|

1: The block timing is affected by the [DoCommand timing caveat](../../Battle%20flow/GetBlock.md#known-issues-with-docommand)

### Logic sequence

- `Blosh` sound plays
- animstate set to 113
- Camera moves to look at this enemy, but zoomed in
- Yield for 1 second
- `Clomp` sound plays
- Yield for 0.75 seconds
- animstate set to 104
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Mist` sound plays
- `poisonsmoke` particles plays near this enemy
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the `TappingKey` command's help)
- DoCommand 1 call happens
- All player party member whose `hp` is above 0 has their y `spin` set to 15.0
- Yield all frames until `doingaction` goes to false. Before each frame yield, all player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`) (but if `blockcooldown` is above 0 meaning a block was done, it's set to 24 (`Block`) instead)
- All player party member whose `hp` is above 0 has DoDamage 1 call happen on them followed by their `spin` getting zeroed out
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- Yield for 0.5 seconds

## Move 4 - Poison bubble throw
A Single target poison bubble throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Happens once if GetAlivePlayerAmmount returns higher than 0 (not all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten). If that is still the case after and `hp` is less than `maxhp` / 2 floored, it happens a second time|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new instance of `Prefabs/Objects/PoisonBubble` childed to this enemy's [model](../../../Entities/EntityControl/Notable%20methods/AddModel.md)'s first child's first child|0.0175 (~57.1429 frames of movement) if hardmode is false, 0.02 if it's true (50.0 frames of movement)|0.0|null|`PoisonEffect`|`BubbleBurst`|null|Vector3.zero|false|

### Logic sequence

- animstate set to 109
- `Blosh` sound plays
- Camera moves to look at this enemy, but zoomed in
- Yield for 1.25 seconds
- Camera moves to look near the center between this enemy and `playerdata[0]`
- animstate set to 111
- `Wub` sound plays on loop using `sounds[9]` with 0.7 pitch
- The amount of bubbles to throw is determined which depends on if `hp` is less than `maxhp` / 2 floored. If it's not, it's 1 bubble thrown and if it is, it's 2 bubbles
- An array of GameObject is initialised to be the amount of bubbles to throw
- For each bubble to throw in the GameObject array:
    - If GetAlivePlayerAmmount returns higher than 0 (not all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten):
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
        - The element of the bubbles array is initialised to a new instance of `Prefabs/Objects/PoisonBubble` childed to the [model](../../../Entities/EntityControl/Notable%20methods/AddModel.md#addmodel)'s first child's first child whose SpriteRender has a material's renderQueue of 10000
        - animstate set to 111
        - `Clomp` sound plays
        - Projectile 1 call happens
        - Yield for 0.5 seconds
        - If this is the last bubble to throw, animstate set to 109 followed by `Blosh` sound plays followed by Yield for 0.5 seconds
    - Otherwise:
        - The element of the bubbles array is reinitialised to a new GameObject
        - Yield for a frame
        - The element of the bubbles array is destroyed
- Yield all frames until all elements of the bubbles array goes to null (meaning they are fully destroyed)
- `sounds[9]` stopped
- Yield for 0.5 seconds

## Move 5 - Drop down from the air
Drops down to `Ground`. No damages are dealt, only the [position](../../Actors%20states/BattlePosition.md) changes to `Ground`.

This move is only used when `data[0]` was 1 at the start of the action while this enemy was `Flying`. Move 1 or 3 will be used right after this one without ending the actor turn.

Only logic changes are mentioned as the rest are only animation related.

### Logic sequence

- Yield for 0.2 seconds
- `data[0]` set to -1
- animstate set to 0 (`Idle`)
- The startstate local is set to 0 (`Idle`)
- Yield for 0.1 seconds
- `height` changes towards 0.0 over the course of 20.0 frames
- `height` set to 0.0
- Yield for 0.6 seconds
- [position](../../Actors%20states/BattlePosition.md) set to `Ground`
