# `Zasp`

## Assumptions
This enemy is meant to function with a second one due to their [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) logic that changes when they are the only one left in `enemydata`. It is meant that this enemy's [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) has the id of this second enemy. Under normal gameplay, this should be [Mothiva1](Mothiva1.md).

## [StartBattle](../../StartBattle.md#startbattle) special logic
If [flags](../../Flags%20arrays/flags.md) 606 is true (won the second round of the Colosseum), the following happens on this enemy:

- `hp` and `maxhp` are increased by 15
- `hardatk` is incremented
- `lockrelayreceive` is set to true

This happens AFTER [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) was called to load this enemy.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This value when 1, was supposed to prevent to use the taunt move again after it had been used in earlier in the same main turn. It is only set to 1 at the end of using the taunt move. However, it is always set to 0 after using any move or `hitaction` meaning it never performs an action without being set to 0 so it effectively cancels out the set to 1 if it happened. This means that this value does nothing and this enemy can use their taunt move multiple times in a row or even in one main turn if hardmode is true

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true has 2 changes:

- The strike attack move can hit from 1 to 3 times instead of 1 to 2 times
- The taunt move will always decrement `cantmove` which gives this enemy an additional actor turn after this one

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves

### `dontusecharge` set to true
The `hitaction` logic always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence
The following logic is performed instead of any moves:

- animstate set to 111
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called with type 2 (red !) for 35 time
- `Wam` sound plays
- Yield all frames until `emotioncooldown` expires
- animstate set to 106
- `StatUp` sound plays
- If this is the only enemy left in `enemydata`:
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 999999 turns
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 0 (red up arrow)
    - `BugWingFast` sound plays
- Otherwise, if `charge` is less than 1:
    - `charge` is incremented
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 4 (green up arrow)
- Yield for 0.65 seconds
- `data[0]` set to 0

## Move selection
4 moves are possible:

1. A single target warp strike attack done multiple times
2. A single target needle throw done multiple times
3. A single target taunt inflicting the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition. This move may continue to the next enemy loop iteration
4. Uses a `VitalitySeed` item to get the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition. This move may continue to the next enemy loop iteration

Here are the odds to select each move per enemy loop iteration:

|Move|Odds|
|---:|----|
|1|4/12|
|2|3/12|
|3|2/12|
|4|3/12|

However, move 3 and 4 may deny the usage of the move and effectively "reroll" the move generation by continuing to the next enemy loop iteration.

## Post move logic
After using any move, the folowing is always done:

- `sprite` enabled
- `overrideanim` set to false
- `rigid` has its gravity enabled without kinematic
- `onground` set to true
- `data[0]` set to 0

## Move 1 - Warp strike attacks
A single target warp strike attack done multiple times

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|GetAlivePlayerAmmount returns higher than 0 (not all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten). Done up to 2 times (up to 3 times if hardmode is true)|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- `specialanim` set to a new disappearing ZaspWarp coroutine
- Yield all frames until `specialanim` goes to null (the coroutine completed)
- The amount of hits is generated being uniformly random between 1 and 2 (between 1 and 3 instead if hardmode is true)
- For each hits when GetAlivePlayerAmmount returns higher than 0 still:
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - An offset vector is generated as follows (each has 50% chance):
        - Upper strike (-2.025 or 2.025 determined with a 50/50, 0.5, -0.1)
        - Lower strike (-2.25 or 2.25 determined with a 50/50, 0.23, -0.1)
    - `specialanim` set to a new appearing ZaspWarp coroutine with a pos of `playertargetentity` + the offset vector
    - Yielf a frame
    - [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) `playertargetentity`
    - Yield all frames until `specialanim` goes to null (the coroutine completed)
    - animstate set to 102 if it was an upper strike, 107 if it was a lower strike
    - `Toss3` sound plays
    - Yield for 0.15 seconds
    - DoDamage 1 call happens
    - Yield for 0.5 seconds
    - `specialanim` set to a new disappearing ZaspWarp coroutine
    - Yield all frames until `specialanim` goes to null (the coroutine completed)
- `specialanim` set to a new appearing ZaspWarp coroutine with a pos of startp
- Yield a frame
- `flip` set to false
- Yield all frames until `specialanim` goes to null (the coroutine completed)

## Move 2 - Needle throws
A single target needle throw done multiple times

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|GetAlivePlayerAmmount returns higher than 0 (not all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten). Done up to 3 times|2|null|This enemy|`playertargetID`|A new GameObject named `needle` with a SpriteRenderer using the `projectilepsrites[2]` (needle sprite)|0.04 (25 frames of movement)|0.0|null|null|null|null|Vector3.zero|false|

### Logic sequence

- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 2 (`Jump`)
- `Spin` sound plays on loop using `sounds[9]`
- `Woosh` sound plays
- Over the course of 10.0 frames, position is lerped to startp + 3.5 in y
- The amount of hits is generated being uniformly random between 1 and 3
- For each hits when GetAlivePlayerAmmount returns higher than 0 still:
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `sounds[9]` volume set to MainManager.`soundvolume`
    - position offset by (-0.35, 1.15, 0.0)
    - animstate set to 101
    - z `spin` set to 30.0
    - Yield for 0.5 seconds
    - `sounds[9]` volume set to 0.0
    - animstate set to 102
    - position set to the one before this hit
    - `spin` zeroed out
    - `sprite` angles reset to Vector3.zero
    - Yield for 0.15 seconds
    - A new GameObject called `needle` is created rooted with a SpriteRenderer using the `projectilepsrites[2]` sprite (needle) with -50.0 z angle positioned at this enemy position + (-1.0, 1.0, 0.1)
    - `Toss` sound plays
    - Projectile 1 call happens
    - Yield all frames until `needle` is null (destroyed as the Projectile coroutine completed)
- animstate set to 101
- z `spin` set to 20.0
- Over the course of 15.0 frames, position lerped to startp + 3.5 in y
- `sounds[9]` stopped
- position set to startp
- animstate set to 103
- `spin` zeroed out
- `sprite` angles reset to Vector3.zero
- Yield for 0.45 seconds

## Move 3 - Taunt
A single target taunt inflicting the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition. No damages are dealt.

### Enemy loop iteration
Instead of the move being performed, the action will continue to the next enemy loop iterations if any of the following conditions are true:

- `data[0]` is above 1 (this is never the case, see the `data` usage section above for why)
- This enemy already has the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition
- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.2

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action)

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 108
- `Taunt2` sound plays on loop using `sounds[9]`
- Yield for 0.85 seconds
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on `playerdata[playertargetID]` for 2 turns (effectively 1 since the main turn gets advanced soon)
- `Taunt` sound plays
- animstate of `playertargetentity` set depending on `playertargetID` (their `trueid`)
    - 0 (`Bee`): 10 (`Flustered`)
    - 1 (`Beetle`): 5 (`Angry`)
    - 2 (`Moth`): 102
- `Prefabs/Particles/AngryParticle` instantiated childed to `playertargetentity` with local position being Vector3.up
- Yield for 0.85 seconds
- `sounds[9]` stopped
- animstate set to 8 (`Happy`)
- Yield for 0.85 seconds
- `Prefabs/Particles/AngryParticle` destroyed
- `playertargetentity` animstate restored to its value before this action
- If hardmode is true, `cantmove` is decremented (gives an additional actor turn to this enemy immediately)
- `data[0]` set to 1 (this does effectively nothing, see the `data` usage section for why)

## Move 4 - Uses a `VitalitySeed` item
Uses a `VitalitySeed` item to get the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition. This move may continue to the next enemy loop iteration. No Damages are dealt.

### Enemy loop iteration
Instead of the move being performed, the action will continue to the next enemy loop iterations if any of the following conditions are true:

- This enemy already has the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.2
- [HPPercent](../../Actors%20states/HPPercent.md) is more than 0.65

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action)

### Logic sequence

- animstate set to 4 (`ItemGet`)
- `ItemHold` sound plays
- A new GameObject with a SpriteRenderer is created rooted using the `VitalitySeed` item sprite positioned near this enemy
- Yield for a second
- The item GameObject is destroyed
- animstate set to 106
- `Magic` sound plays
- y `spin` set to 20.0
- Yield for 0.75 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 0 (red up arrow)
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 3 turns (effectively 2 since the main turn gets advanced soon)
- `spin` zeroed out
- Yield for 0.75 seconds
