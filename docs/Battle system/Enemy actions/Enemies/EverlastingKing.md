# `EverlastingKing`

## Assumptions
It is assumed this enemy is fought alone as otherwise, it would break theor artifact summoning logic.

It is also assumed this enemy is loaded with `SurviveWith10` in their [weakness](../../Actors%20states/Enemy%20features.md#weakness) as it allows their phase transition logic to work as expected.

It is also assumed this enemy's [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) to be configured to use the [matching EventDialogue](../../Battle%20flow/EventDialogues/EverlastingKing.md) as it allows them to die with a mini cutscene and end the battle properly.

It is also assumed this enemy is loaded with [actimmobile](../../Actors%20states/Enemy%20features.md#actimmobile) set to true which allows them to cure themseolves of [stopping condition](../../Actors%20states/IsStopped.md).

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 5 element with a starting value of 0.

- `data[0]`: A cooldown until this enemy uses the [position](../../Actors%20states/BattlePosition.md) change to `Flying` move. If this enemy's `position` is `Ground`, the value is decremented before deciding which of the 2 attacking `Ground` moves. If after this decrement, the value becomes -3, this enemy will use the `position` change move so it will go to `Flying` which also causes this value to be reset to 0. Effectively, it means that at most, this enemy will remain on `Ground` for 2 actor turns in a row before using the `position` change move. NOTE: Since the value is only decremented when their `position` is `Ground`, if the player causes them to change to `Flying`, it freezes this cooldown, but it will apply still the moment this enemy comes back to `Ground`
- `data[1]`: A value that tracks the last move that was used when this enemy's [position](../../Actors%20states/BattlePosition.md) was `Flying`. This is used as an antispam on the fire pillar attack move, the artifact summoning move and the combined [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) attack move. See the move selection process section below for more details on how this is enforced. The value is set in the post move logic so it doesn't track the moves used when on `Ground` and it also means that the value stays the same as long as `position` is `Ground`. Here are the different possible values and their mappings:
    - 0: Vine attack move (`Flying` version)
    - 1: Fire pillar attack move
    - 2: Axe throw move
    - 3: Artifact summoning move
    - 4: Double fireballs throw move
    - 5: Combined [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) attack move
- `data[2]`: A phase tracker. It can be only one of the following values:
    - 0: The initial state and first phase. It emans this enemy hasn't underwent any phase transition. Once the first phase transition happens when this enemy's `hp` is 10 or lower in the pre move logic, the value is set to 1
    - 1: The second phase. It means this enemy went through the first phase transition. Once the second phase transition when this enemy's `hp` is 10 or lower in the pre move logic once again, the value is set to 2
    - 2: The third and last phase. It means this enemy went through both phase transitions. From now on, this value cannot influence any logic
- `data[3]`: A cooldown that when expires, it overrides the `Flying` move selection process to select the artifact summoning move (but it might not necessarily use it, see the move selection process for details). It has a starting value of 2 on the first main turn. When using the artifact summoning move, the value is set to 3 (which will become 2 by the end of the actor turn due to the post move logic). In the post move logic (meaning it only applies when `Flying`), the value is decremented if it is above 0. During the `Flying` move selection process, if the value is 0, the artifact summoning move will always be selected and the value will be set to 2. This can have unexpected interactions with `data[4]`, see the move selection process for more details. Effectively, if other conditions are satisfied, it means that this enemy will be using the move after 2 actor turns they spent while `Flying`. NOTE: the value is set to 0 when the first phase transition occurs in the pre move logic and it is reset to 2 when the second phase transition occurs in the pre move logic
- `data[4]`: A cooldown on the usage of the artifact summoning move. When the move is used, the value is set to 2 when summoning a [KeyR](KeyR%20and%20KeyL.md) or to 3 when summoning a [Tablet](Tablet.md). In the post move logic (meaning it only applies when `Flying`), the value is decremented if it is above 0. For the artifact summoning move to be come usable again, the value must be 0 by the time the `Flying` move selection process happens, even if `data[3]` reached 0 causing a selection override (see the move selection section below for details on how this can interact with `data[3]`). Effectively, it means that this enemy needs to spend 1 or 2 actor turns while `Flying` before being able to use the move again. NOTE: the value is set to 5 when the second phase transition occurs in the pre move logic meaning this enemy will need to spend 5 actor turns while flying before being able to use the move again

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- When summoning any artifact enemies (whether by using the summoning move or as part of the second phase transition), the summoned enemy's `cantmove` is now decremented meaning they can act immediately on the same main turn they were summoned
- During the `Ground` move selection process, the odds of which move to use changes to favor the vine attack move (`Ground` version)
- In the vine attack move (`Ground` version), the move may hit 2 or 3 times instead of only once
- In the flower bud attack move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the vine attack move (`Flying` version), the move may hit twice instead of only once
- In the axe throw move, the amount of frames each axe throw changes to 21.0 frames from 30.0 frames on the first throw and to 17.0 frames from 21.0 frames on the second throw
- In the double fireballs throw move, the speed of each Projectile calls changes to be random between 22.0 and 28.0 instead of being Random between 29.0 and 35.0
- In the combined [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) attack move, the damageammount of the DoDamage call changes to 9 from 10

Also, on HARDEST specicifically, this enemy always heals their `hp` to full on each of the 2 phase transition instead of being a fraction of their `maxhp`.

## Move selection
9 moves are possible:

1. A single target vine attack (`Ground` version) that may hit multiple times followed by a 3 `hp` heal
2. A single target flower bud attack that continuously drains `hp` followed by a 3 `hp` heal
3. Changes their [position](../../Actors%20states/BattlePosition.md) to `Flying` and gets another actor turn
4. A single target vine attack (`Flying` version) that may hit multiple times
5. A party wide fire pillar attack
6. A single target axe throw attack that may hit twice
7. Summons an enemy from the artifact family which is composed of [KeyL](KeyR%20and%20KeyL.md), [KeyR](KeyR%20and%20KeyL.md) and [Tablet](Tablet.md)
8. A multiple targets double fireballs throws
9. A single target attack combining [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) into a laser slash attack

There's 2 move selection process depending on [Position](../../Actors%20states/BattlePosition.md): one for `Ground` and the other for `Flying` and they affect different moves while functioning mostly independently from each other.

### `Ground` move selection process
This selection process only applies to move 1, 2 and 3. Those move are only usable when `position` is `Ground`.

Move 3 is always (and only) used when `data[0]` is -3 or lower after a decrement (meaning the countdown to stay on `Ground` expired). This move is a way to transition to the `Flying` move selection process since this enemy always gets another actor turn when using that move.

As for move 1 and 2, the decision of which move to use is based on odds, but the odds changes depending if hardmode is true or false. Here are the odds in either situations:

|Move|Odds when hardmode is true|Odds when hardmode is true|
|---:|------|-----|
|1|40%|60%|
|2|60%|40%|

### `Flying` move selection process
This selection process only applies to move 4, through 9. Those move are only usable when `position` is `Flying`.

The decision of which move to use is based on odds, but some moves have additional requirements that needs to be fufilled in order for the move to be used once selected. If the requirements aren't fufilled after selecting the move, a continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection process. Here are all the odds and requirements of each moves:

|Move|Odds|Requirements|
|---:|----|-----------|
|4|3/24|None, the move is always used once selected|
|5|2/24|`data[1]` isn't 1 (meaning this move wasn't used on the last actor turn)|
|6|3/24|None, the move is always used once selected|
|7|6/24|<ul><li>There's less than 4 `enemydata`</li><li>`data[1]` isn't 3 (meaning this move wasn't used on the last actor turn)</li><li>`data[4]` is 0 (the cooldown on the usage of this move expired)</li></ul>|
|8|3/24|None, the move is always used once selected|
|9|7/24|<ul><li>`data[1]` isn't 3 (meaning move 7 wasn't used on the last actor turn)</li><li>A [KeyL](KeyR%20and%20KeyL.md) is present in `enemydata`</li><li>A [KeyR](KeyR%20and%20KeyL.md) is present in `enemydata`</li></ul>|

#### Move 7 override case
Move 7 has is handled in a special way on top of the table above. It is possible that the initial move selection is overriden to select move 7 if all of the following conditions are fufilled:

- `data[3]` is 0 or lower (the countdown to force the usage of move 7 expired)
- There's less than 4 `enemydata`
- `data[1]` isn't 3 (move 7 wasn't used on the last actor turn)

When this happens, `data[3]` is set to 2 and move 7 is always selected. However, this only guarantees the SELECTION of the move: the move is still subject to its requirements in order to actually be used. It means that there's a condition that can still cause the move to be selected like this, but not be used and that is `data[4]` could still be above 0 (meaning the cooldown on the usage of the move hasn't expired yet), but all other requirements would be met.

Since `data[4]` isn't checked before applying this selectioon override, it's possible the override happens, but the move won't be used and another will be instead due to the continue directive. Since this override case sets `data[3]` to 2, it can't happen again once the continue directive is issued.

Effectively, it means it's possible the move 7 override misses its chance to actually force the usage of the move depending on if `data[4]` is above 0 or not at the time the override case applies.

## SummonArtifact
There is a coroutine specific to this enemy that is used everytime an artifact enemy needs to be summoned as it involved specialised logic. It is documented here for reference simplicity. It is meant to be set to `enemybounce[0]` as it will be set to null when the coroutine completed.

```cs
private IEnumerator SummonArtifact(int calledby, int type, bool hardmode)
```

### Parameters

- `calledby`: The `enemydata` index of this enemy
- `type`: An enemy id that can either be 101 ([KeyR](KeyR%20and%20KeyL.md)), 102 ([KeyL](KeyR%20and%20KeyL.md)) or 103 ([Tablet](Tablet.md)). If the value is -1, it means to randomly select one that isn't present randomly with uniform odds
- `hardmode`: A way for the method to get the hardmode value

### Procedure

- This enemy's `enemydata` is obtained via calledby
- `data[3]` is set to 3
- The enemy id is generated based on the type paramneter described above
- `EverLastingKingSummonArtifact` sound plays
- Yield for a second
- Yield for 0.5 seconds
- animstate set to 105
- Based on the enemy that will be summoned, the position of the summon and `data[4]` value (the cooldown on the usage of the artifact summoning move) are determined:
    - `KeyL`: summoned at (1.5, 0.0, 0.5) with no `data[4]` changes
    - `KeyR`: summoned at (5.0, 0.0, -0.85) with `data[4]` set to 2
    - `Tablet`: summoned at (2.0, 0.0, -0.25) with `data[4]` set to 3
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon the selected enemy at the position determined without cantmove (meaning the enemy will be able to act on the same main turn). As for the type of the summon, it's `Offscreen` except for the `Tablet` where it's the `Tablet` type
- Yield all frames until `checkingdead` is null (the coroutine completed)
- This enemy animstate set to 106
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- The newly summoned enemy's [diebyitself](../../Actors%20states/Enemy%20features.md#diebyitself) is set to true
- If hardmode is true, this enemy's `cantmove` is decremented which grants them an additional actor turn (this is meant to be undone if called from a phase transition)
- `enemybounce[0]` set to null which signals the caller that this coroutine completed

## Pre move logic
The following logic happens before the usage of any moves. It's in multiple parts done in order.

### [Position](../../Actors%20states/BattlePosition.md) adjustement
This part always happen and it causes the `position` to be set depending on `height`:

- If it's 0.5 or lower, it's set to `Ground`
- If it's higher than 0.5, it's set to `Flying`

### Stop condition curing
This part only happens if this enemy [IsStopped](../../Actors%20states/IsStopped.md) with skipimmobile:

- `Charge7` sound plays with 0.8 pitch
- ShakeScreen called with an ammount of 0.1 and 0.5 time with dontreset
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on this enemy
- `Heal3` sound plays
- Yield for 0.5 seconds 

### `hologram` adjustements
This part only happens if `hologram` is true and the only thing that happens is the `sprite`'s material's renderQueue is set to 2500 which allows the shielding effect to render correctly visually.

### Phase transition
This part only happens if `hp` is 10 or lower and `data[2]` is less than 2 (meaning this enemy hasn't gone through both phase transitions yet).

- If `hologram` is false:
    - Camera moves to look near (this enemy x position, `sprite` local y position + 0.25, 2.5)
    - Yield for 0.5 seconds
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[175]` when `data[2]` is 0 (first phase) or `commondialogue[176]` when `data[2]` is 1 (second phase)
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: This enemy
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - Yield for 0.25 seconds
- animstate set to 100
- ShakeSprite called with 0.2 intensity and 30.0 frametimer
- `Rumble` sound plays
- ShakeScreen called with 0.1 ammount and 2.0 time with dontreset
- Done in a separate coroutine started in paralel called MainManager.Waves:
    - A `Prefabs/Objects/SphereGlowEffect` GameObject is instantiated rooted at this enemy's world [CenterPos](../../Actors%20states/CenterPos.md) with a scale of (50.0, 50.0, 1.0) with a pure green color
    - Over the course of 121.0 frames, the scale of `Prefabs/Objects/SphereGlowEffect` changes to 0.0x via a lerp
    - The `Prefabs/Objects/SphereGlowEffect` gets destroyed
- `EverLastingKingVine` sound plays with 0.8 pitch
- Yield for a second
- Yield for a second
- `Rumble` sound plays
- The amount of `hp` to set this enemy is determined:
    - If [flags](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active), It's all their `hp`
    - If `data[2]` is 0 (first phase), it's their `maxhp` * 0.9 floored. The amount is substracted by 5 if their [position](../../Actors%20states/BattlePosition.md) is `Ground`
    - If `data[2]` is 0 (first phase), it's their `maxhp` * 0.75 floored. The amount is substracted by 5 if their [position](../../Actors%20states/BattlePosition.md) is `Ground`
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by the determined amount of `hp`. NOTE: this is technically missleading because it is healing by the set amount - 10, it isn't healing by the amount determined as that's the amount the `hp` will be SET AT, not the amount HEALED BY
- If [flags](../../../Flags%20arrays/flags.md) 614 is false (HARDEST is inactive), `hp` is set to the amount determined
- Yield for 0.5 seconds
- `data[3]` is set to 0 (the move selection override case is set to happen on this actor turn if all conditions are fufilled)
- If `hologram` is false:
    - Every player party member whose `hp` is 0 gets revived via [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) leaving them with 1 `hp` without showcounter
    - [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called
    - Yield for 0.5 seconds
    - If `data[2]` is 0 (first phase), 2 [SetText](../../../SetText/SetText.md) occurs in [dialogue mode](../../../SetText/Dialogue%20mode.md) in a row (yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock released after each calls). The first one has `playerdata[0]` (`Bee`) as the parent with `commondialogue[199]` as the text and the second has `playerdata[1]` (`Beetle`) as the parent with `commondialogue[200]` as the text
    - Otherwise (`data[2]` is 1 which is the second phase), 2 [SetText](../../../SetText/SetText.md) occurs in [dialogue mode](../../../SetText/Dialogue%20mode.md) in a row (yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock released after each calls). The first one has `playerdata[1]` (`Beetle`) as the parent with `commondialogue[201]` as the text and the second has `playerdata[2]` (`Moth`) as the parent with `commondialogue[202]` as the text
    - Yield for 0.5 seconds
- If `data[2]` is 1 (performing the last phase transition):
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - `SurviveWith10` is removed from this enemy [weakness](../../Actors%20states/Enemy%20features.md#weakness) which makes them vulnerable to be killed
    - `enemybounce` set to a one element Coroutine array
    - `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call to change [position](../../Actors%20states/BattlePosition.md) to `Flying`
    - Yield all frames until `checkingdead` is null (the coroutine completed)
    - If [KeyL](KeyR%20and%20KeyL.md) isn't present:
        - `enemybounce[0]` is set to a new SummonArtifact call to summon a new `KeyL` enemy
        - Yield all frames until `enemybounce[0]` is null (the coroutine completed)
    - If [KeyR](KeyR%20and%20KeyL.md) isn't present:
        - `enemybounce[0]` is set to a new SummonArtifact call to summon a new `KeyR` enemy
        - Yield all frames until `enemybounce[0]` is null (the coroutine completed)
    - `data[4]` is set to 5 (meaning the artifact summoning move won't be usable for the next 5 actor turns spent while `Flying`)
    - `cantmove` is set to 0 (this undo the actor turn granted by SummonArtifact calls as they are undesired here)
    - `data[3]` is reset to 2
- `data[2]` is incremented

## Post move logic
The following logic applies after a move is used, but only if [position](../../Actors%20states/BattlePosition.md) is `Flying` meaning it only applies when the `Flying` move selection process happened and a move was used.

If it's any move other than these, the following happens:

- `data[1]` is set to a value that tracks the move that was just used. See the `data` section above for the values mapping
- If the move that was just used isn't the artifacts summoning move:
    - If `data[3]` is above 0, it is decremented (this is the countdown to force the selection of the artifacts summoning move)
    - If `data[4]` is above 0, it is decremented (this is the cooldown for the usage of the artifacts summoning move)

## Move 1 - Vine attack while on `Ground`
A single target vine attack (`Ground` version) that may hit multiple times followed by a 3 `hp` heal.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|`playerdata[playertargetID]`'s `hp` is above 0. Done once if hardmode is false, from 2 to 3 times if it is true as long as `playerdata[playertargetID]`'s `hp` is above 0|This enemy|The selected `playertargetID`|5 on the first hit, 2 on the second hit and 1 on the third hit|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- animstate set to 109
- `EverLastingKingVine` sound plays
- Yield for 0.5 seconds
- `checkingdead` is set to a new VineAttack coroutine with 5 damageamt, actionid as the callerid with gettarget. The coroutine will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` goes to null
- animstate set to 0 (`Idle`)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Yield for 0.25 seconds
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by 3 `hp`
- Yield for 0.5 seconds

This is what the coroutine effectively ends up doing:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Rumble3` sound plays
- ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
- Camera moves to look near `playertargetentity`
- The amount of hits is determined. It is 1 if hardmode is false and if it's true, it's random between 2 and 3 inclusive
- An array of `Prefabs/Objects/VenusRoot` are instantiated with the amount being the amount of hits to do at (`playertargetentity` x position + a random number between -2.0 and 2.0, -5.0, `playertargetentity` z position). Each instance is set to LookAt the `playertargetentity` and has its initial position incremented by (random between -1.0 and 1.0, 0.0, -0.15)
- An array of Vector3 are instantiated with the amount being the amount of hits to do with each being the normalised direction vector from `playertargetentity` to the matching `Prefabs/Objects/VenusRoot` element
- Yield for 0.75 seconds
- For each hit:
    - `Charge8` sound plays
    - The matching `Prefabs/Objects/VenusRoot` scale is multiplied by 1.65
    - Over the course of 10.0 frames, the matching `Prefabs/Objects/VenusRoot` position lerps to its position + the matching heading direction vector * 5.0. The first time reaching the 5th frame or later when `playerdata[playertargetID]`'s `hp` is above 0, DoDamage 1 call happens before the frame yield
    - Yield for 0.5 seconds
    - Yield for 0.5 seconds
    - If this is the last hit:
        - `Rumble3` sound plays
        - ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
        - Yield for 0.5 seconds
    - The amount of damage to deal on the next hit becomes half of this one's damage floored then clamped from 1 to 99
- Over the course of 60.0 frames, all `Prefabs/Objects/VenusRoot` instances has ther scale lerped to Vector3.zero
- Yield a frame
- All `Prefabs/Objects/VenusRoot` instances gets destroyed
- `checkingdead` set to null which informs the caller that the coroutine is done

## Move 2 - Flower bud attack
A single target flower bud attack that continuously drains `hp` followed by a 3 `hp` heal.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|-1.0 (infinite)|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Called once if `barfill` is less than 1.0 (not full) during DoCommand 1 once 0.5 seconds passed and then continously called again every 1.0 seconds until either `barfill` reaches 1.0 (checked 1.0 seconds after each call) or that `playerdata[playertargetID]`'s `hp` becomes 0 (checked after each call)|This enemy|The selected `playertargetID`|1|[Atleast1pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|false|

1: Enemy piercing damages are disabled, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity` position
- animstate set to 109
- Yield for 0.5 seconds
- `Rumble3` sound plays
- ShakeScreen called with 0.05 ammount and -1.0 time (infinite)
- Yield for 0.5 seconds
- ShakeScreen called with 0.2 ammount and 0.75 time with dontreset
- A `Prefabs/Objects/OpeningFlowerBud` GameObject is instantiated rooted at `playertargetentity` position - 0.1 in z whose SpriteRenderer has a material of MainManager.`spritemat` and a scale of 2.0x
- `playertargetentity` animstate set to 11 (`Hurt`)
- `Spin9` sound plays with 0.8 pitch
- `DirtExplode` particles plays at `playertargetentity` position
- The `Close` animation clip is played on `Prefabs/Objects/OpeningFlowerBud`'s Animator
- `Dig3` sound plays
- animstate set to 0 (`Idle`)
- `Clomp` sound plays
- Yield for 0.5 seconds
- A SpriteBounce is added to the `Prefabs/Objects/OpeningFlowerBud` with a MessageBounce of 2.0 multiplier and with `startscale` set to true
- DoCommand 1 call happens
- `infinitecommande` set to true which will prevent DoCommand 1 to end when MainManager.`mashcommandalt` is true until the action command is succeeded
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the [TappingKey](../../Action%20commands/TappingKey.md) command's help)
- As long as `playerdata[playertargetID]`'s `hp` is above 0 and `actionroutine` isn't null (meaning DoCommand 1 hasn't ended yet):
    - Yield for 0.5 seconds
    - If `barfill` is less than 1.0 (it isn't full yet):
        - DoDamage 1 call happens
        - If `lastdamage` is above 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy by `lastdamage` amount of `hp`
    - Yield for 0.75 seconds
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- `Barfill` sound stops
- If `playerdata[playertargetID]`'s `hp` is above 0:
    - Their animstate is set to the value it had before this action
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on them
- FinishAction is called manually to end DoCommand 1 if it wasn't already (but it should be by then)
- The `Open` animation clip is played on `Prefabs/Objects/OpeningFlowerBud`'s Animator
- `CherryBoom` particles plays at the `playertargetentity` position
- ShakeScreen called with 0.1 ammount and 0.5 time with dontreset
- MainManager.MoveTowards called to move `Prefabs/Objects/OpeningFlowerBud` to - 2.0 in y with a 30.0 multiplier without smooth and without local (done in paralel)
- Yield for 0.5 seconds
- The `Prefabs/Objects/OpeningFlowerBud` gets destroyed in 2.0 seconds
- animstate set to 0 (`Idle`)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Yield for 0.25 seconds
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by 3 `hp`
- Yield for 0.5 seconds

## Move 3 - Changes [position](../../Actors%20states/BattlePosition.md) to `Flying` with another actor turn
Changes their [position](../../Actors%20states/BattlePosition.md) to `Flying` and gets another actor turn. No damages are dealt, but this enemy always gets another actor turn which may deal damages.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect anything for this move.

### Logic sequence

- `data[0]` is reset to 0 (the cooldown before using this move when [position](../../Actors%20states/BattlePosition.md) is `Ground`)
- `checkingdead` set to a new [ChangePosition](../ChangePosition.md) calle to change the [position](../../Actors%20states/BattlePosition.md) of this enmey to `Flying`
- Yield all frames until `checkingdead` is null (the coroutine compelted)
- `cantmove` is decremented which grants another actor turn to this enemy (it transitions to the `Flying` move selection process)

## Move 4 - Vine attack while `Flying`
A single target vine attack (`Flying` version) that may hit multiple times.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|`playerdata[playertargetID]`'s `hp` is above 0. Done once if hardmode is false, from 1 to 2 times if it is true as long as `playerdata[playertargetID]`'s `hp` is above 0|This enemy|The selected `playertargetID`|3 on the first hit, 1 on the second hit (4 on the first hit and 2 on the second hit instead if `data[2]` is 2 meaning this enemy went through both phase transitions)|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- animstate set to 101
- `EverlastingKingVine` sound plays
- Yield for 0.5 seconds
- `checkingdead` is set to a new VineAttack coroutine with 3 damageamt (4 instead if `data[2]` is 2 meaning this enemy went through both phase transitions), actionid as the callerid with gettarget. The coroutine will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` goes to null

This is what the coroutine effectively ends up doing:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Rumble3` sound plays
- ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
- Camera moves to look near `playertargetentity`
- The amount of hits is determined. It is 1 if hardmode is false and if it's true, it's random between 1 and 2
- An array of `Prefabs/Objects/VenusRoot` are instantiated with the amount being the amount of hits to do at (`playertargetentity` x position + a random number between -2.0 and 2.0, -5.0, `playertargetentity` z position). Each instance is set to LookAt the `playertargetentity` and has its initial position incremented by (random between -1.0 and 1.0, 0.0, -0.15)
- An array of Vector3 are instantiated with the amount being the amount of hits to do with each being the normalised direction vector from `playertargetentity` to the matching `Prefabs/Objects/VenusRoot` element
- Yield for 0.75 seconds
- For each hit:
    - `Charge8` sound plays
    - Over the course of 10.0 frames, the matching `Prefabs/Objects/VenusRoot` position lerps to its position + the matching heading direction vector * 5.0. The first time reaching the 5th frame or later when `playerdata[playertargetID]`'s `hp` is above 0, DoDamage 1 call happens before the frame yield
    - Yield for 0.5 seconds
    - If this is the last hit:
        - `Rumble3` sound plays
        - ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
        - Yield for 0.5 seconds
    - The amount of damage to deal on the next hit becomes half of this one's damage floored then clamped from 1 to 99
- Over the course of 60.0 frames, all `Prefabs/Objects/VenusRoot` instances has ther scale lerped to Vector3.zero
- Yield a frame
- All `Prefabs/Objects/VenusRoot` instances gets destroyed
- `checkingdead` set to null which informs the caller that the coroutine is done

## Move 5 - Fire pillar attack
A party wide fire pillar attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Fire](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 101
- `checkingdead` set to a new FirePillar coroutine from this enemy without green
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine ends up doing:

- Yield for 0.5 seconds
- `FirePillar` sound plays
- A new `Prefabs/Objects/FirePillar 1` GameObject is instantiated childed to the `battlemap` at `partymiddle` with a scale of (0.0, 1.0, 0.0 )and a DialogueAnim with the following:
    - `targetscale`: (0.75, 1.0, 0.75)
    - `shrink`: false
    - `shrinkspeed`: 0.015
- Yield for 0.65 seconds
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `shrinkspeed` set to 0.2
- Yield for 0.2 seconds
- ShakeScreen called with ammount of 0.25 and 0.75 time
- PartyDamage 1 call happens
- Yield for 1.15 seconds
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `targetscale` set to (0.0, 1.0, 0.0)
- All ParticleSystem under `Prefabs/Objects/FirePillar 1` are stopped
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `shrinkspeed` set to 0.05
- `Prefabs/Objects/FirePillar 1` gets destroyed
- `checkingdead` is set to null which signals the caller that this coroutine completed

## Move 6 - Axe throw attack
A single target axe throw attack that may hit twice.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|
|2|Happens after DoDamage 1 if the target's `hp` is above 0 and either [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.6 or `data[2]` is 2 (this enemy went through both phase transitions)|This enemy|The same target as DoDamage 1|3|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- `BagRustle` sound plays
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 102
- Yield for a second
- Yield for 0.5 seconds
- animstate set to 103
- A new sprite object is created childed to the `battlemap` using the `Sprites/Entities/waspking2` sprite at index 53 (this enemy's axe) positioned at `sprite` position + (-1.0, 1.0, -0.1)
- If `hologram` is true, the `Sprites/Entities/waspking2` material is set to MainManager.`holosprite`
- `WaspKingAxToss` sound plays
- Over the course of 30.0 frames (21.0 frames instead if hardmode is true), the axe moves to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp. Before each frame yield, the z angle of the axe increases by 30.0 * MainManager.`framestep`
- ShakeScreen called with 0.2 ammount and 0.5 time with dontreset
- DoDamage 1 call happens
- Over the course of 41.0 frames, the axe moves to `sprite` position + (-1.0, 1.0, -0.1) via a BeizierCurve3 with a ymax of 5.0. Before each frame yield, the z angle of the axe increases by 20.0 * MainManager.`framestep`. On the first frame of the second half, if `playerdata[playertargetID]`'s `hp` is above 0 and either `data[2]` is 2 (went through both phase transition) or [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.6, the second hit will happen:
    - animstate set to 101
    - Over the course of 31.0 frames, the z angle of the axe increases by an amount that progressively changes from 0.0 to -30.0 * MainManager.`framestep` which means it will progressively spin the other way in z
    - Yield for 0.1 seconds
    - `WaspKingAxSpin` sound plays
    - Over the course of 30.0 frames, the z angle of the axe increases by an amount that progressively changes from 0.0 to 60.0 * MainManager.`framestep` which means it will progressively spin faster in the original way in z
    - Over the course of 21.0 frames (17.0 frames instead if hardmode is true), the axe moves to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp. Before each frame yield, the z angle of the axe increases by 30.0 * MainManager.`framestep`
    - ShakeScreen called with 0.2 ammount and 0.5 time with dontreset
    - DoDamage 2 call happens
    - The entire axe movement restarts from frame 0, but the second hit won't happen again
- `WaspKingAxeThrowCatch` sound plays
- The axe object gets destroyed
- animstate set to 104
- Yield for 0.5 seconds
- Yield for 0.25 seconds

## Move 7 - Artifact summon
Summons an enemy from the artifact family which is composed of [KeyL](KeyR%20and%20KeyL.md), [KeyR](KeyR%20and%20KeyL.md) and [Tablet](Tablet.md). No damages are dealt, but it is possible this enemy gets another actor turn which may deal damages.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### Logic sequence

- `enemybounce` is set to a new coroutine with 1 element
- `enemybounce[0]` is set a new SummonArtifact called with type -1 meaning any artifact that isn't currently present will be summoned (determined randomly if multiple aren't present)
- Yield all frames until `enemybounce[0]` is null (the coroutine completed)

## Move 8 - Double fireballs throws
A multiple targets double fireballs throws.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3|[Fire](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new `Prefabs/Particles/Fireball` GameObject childed to the `battlemap` positioned at this enemy's `sprite` position + (0.0, 4.0, -0.1) with a scale of 1.0x|Random between 29.0 and 35.0 (random between 22.0 and 28.0 instead if hardmode is true)|2.0|`SepPart@2@4`|`Fire`|`WaspKingMFireball2`|null|Vector3.zero|false|

### Logic sequence

- A new array of 2 transform is initialised which will holds the projectiles objects
- For each hit (always 2 times), if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `WaspKingMFireball1` sound plays
    - animstate set to 107
    - The projectile element is set to a new `Prefabs/Particles/Fireball` GameObject childed to the `battlemap` positioned at this enemy's `sprite` position + (0.0, 4.0, -0.1)
    - Over the course of 36.0 frames, the `Prefabs/Particles/Fireball`'s scale changes from 0.0x to 1.0x via a lerp
    - Yield for 0.25 seconds
    - animstate set to 103
    - `WaspKingFireball2` sound plays
    - Projectile 1 call happens
    - Yield for a random interval between 0.3 and 0.6 seconds
- animstate set to 0 (`Idle`)
- Yield all frames until all projectile objects are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 9 - Combined [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) attack
A single target attack combining [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) into a laser slash attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|10 (9 instead if hardmode is true)|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- Camera moves to look near (this enemy x position, 2.75, 1.0)
- Both [KeyL](KeyR%20and%20KeyL.md) and [KeyR](KeyR%20and%20KeyL.md) present has their `inice` set to true and their `sprite` angles reset to Vector3.zero
- animstate set to 107
- `ArtifactSpin` sound plays
- Over the course of 60.0 frames, `KeyL`'s y `spin` changes to 30.0 via a lerp over the first 30.0 frames and `KeyR`'s y `spin` changes to -30.0 via a lerp over the first 30.0 frames
- Over the course of 31.0 frames, both `KeyL` and `KeyR` moves to this enemy position + (0.0, 6.0, -0.1) - their `height` in y via a lerp
- `ArtifactKey` sound plays with 1.2 pitch
- FadeIn to pure white with 0.2 speed
- Yield for 0.5 seconds
- both `KeyL` and `KeyR` position set to offscreen at -999.0 in y
- A new sprite object is created childed to the `battlemap` using the `Sprites/Objects/artifacts` sprite at index 7 (a full key with both halves together) positioned at this enemy position + (0.0, 6.0, -0.1)
- If `hologram` is true, the key's material is set to MainManager.`holosprite`
- Yield for a frame
- FadeOut with 0.2 speed
- Over the course of 101.0 frames, the key's z angle changes from 2000.0 to 90.0 via a lerp
- Yield for 0.25 seconds
- ShakeScreen called with 0.1 ammount and 0.75 time with dontreset
- `Lazer2` particles plays at Vector3.zero with -1.0 alivetime (infinite) childed to the key with a y angle of 90.0 and a local position of 1.0 in x
- `OmegaEye` sound plays with 1.2 pitch
- `Flash2` sound plays with 1.5 pitch
- Yield for a second
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- Over the course of 51.0 frames, the key moves to `playertargetentity` position + (2.0, 3.5, -0.1) via a SmoothLerp and their angles changes to (0.0, 0.0, 65.0) via a LerpVectorAngle
- `Charge7` sound plays with 1.35 pitch
- MainMansger.ShakeObject called on the key with 0.2 in x as the shake and 45.0 frametimer with returntostart (done in paralel)
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 101
- `Slash3` sound plays
- Over the course of 8.0 frames, the key z angle changes from 65.0 to 300.0 via a lerp. On the first frame after 3.5 frames, DoDamage 1 call happens
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- ShakeScreen called with 0.1 ammount and 0.65 time with dontreset
- The key gets destroyed
- `Spin3` sound plays with 0.75 pitch
- Over the course of 51.0 frames, both `KeyL` and `KeyR` moves from the position of the key when it was destroyed - the `KeyL`'s `height` in y to their respective postion before this action via a BeizierCurve3 with a ymax of 6.0. Also, `KeyL`'s y `spin` changes from 30.0 to 0.0 via a lerp and `KeyR`'s y `spin` changes to -30.0 to 0.0 via a lerp
- Both `KeyL` and `KeyR` has their `inice` set to false and their `spin` zeroed out
