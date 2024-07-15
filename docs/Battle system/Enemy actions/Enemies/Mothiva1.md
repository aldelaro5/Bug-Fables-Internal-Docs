# `Mothiva1`

## Assumptions
This enemy is meant to function with [Zasp](Zasp.md) present as due to their [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) logic that changes when they are the only one left in `enemydata`. This enemy also has logic change if `Zasp` is present or not.

## [StartBattle](../../StartBattle.md#startbattle) special logic
If [flags](../../Flags%20arrays/flags.md) 606 is true (won the second round of the Colosseum), the following happens on this enemy:

- `hp` and `maxhp` are increased by 15
- `hardatk` is incremented
- `lockrelayreceive` is set to true

This happens AFTER [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) was called to load this enemy.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: The amount of times this enemy revived [Zasp](Zasp.md) from `reservedata`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true has 3 changes:

- In the music note throw move (party wide version), the note will take 40 frames instead of 50 frames to reach its target leading to the DoDamage call
- The relay move may be used instead of always being redirected to either the music note throw or the kick move
- When deciding to use the revive move, it is now possible with a 51% chance to redirect to the kick move if `data[0]` is 1 or above. Normally, this would only have been possible if `data[0]` is 2 or above and pass the 51% RNG check

## Move selection
5 moves are possible:

1. A single target that may hit multiple times or party wide music note throw
2. A single target kick attack that may hit multiple times
3. Sings which inflicts the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to [Zasp](Zasp.md)
4. Relay her turn which gives an actor turn to [Zasp](Zasp.md)
5. Revives [Zasp](Zasp.md) from the `reservedata`

Move 5 is special because its decision requires all the following conditions to be true and if they are fufilled, it takes priority over all other moves:

- `forceattack` is -1 (meaning the enemy party isn't affected by [BeetleTaunt](../../Player%20actions/Skills/BeetleTaunt.md))
- `reservedata` exists and contains [Zasp](Zasp.md)
- A 75% RNG check passes

All the other 4 moves have base odds which are shown in the following table. However, the decision of the move to use may not be the final one. It is possible the move's logic decides to perform a different move instead which is reffered to as a redirection. Redirections may chain together and they typically happen when proceeding with the move is illogical or undesired.

Here are all the move's base odds and their possible redirections:

|Move|Base odds|Possible redirections|
|---:|----------|----------------------|
|1|3/7|None|
|2|2/7|Move 1 if [HPPercent](../../Actors%20states/HPPercent.md) is higher than 0.6|
|3|1/7|<ul><li>Move 1 or 2 determined randomly if any of the following is true<sup>1</sup>:<ul><li>`forceattack` isn't -1 (meaning the enemy party is affected by [BeetleTaunt](../../Player%20actions/Skills/BeetleTaunt.md))</li><li>[Zasp](Zasp.md) is not present in `enemydata`</li><li>[Zasp](Zasp.md) is [stopped](../../Actors%20states/IsStopped.md)</li></ul></li><li>Move 1 if the above didn't apply, but [Zasp](Zasp.md) already has the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition<sup>1</sup></li></ul>|
|4|1/7|Move 1 or 2 determined randomly if any of the following is true<sup>1</sup>:<ul><li>hardmode is false</li><li>`forceattack` isn't -1 (meaning the enemy party is affected by [BeetleTaunt](../../Player%20actions/Skills/BeetleTaunt.md))</li><li>[Zasp](Zasp.md) is not present in `enemydata`</li><li>[Zasp](Zasp.md) is [stopped](../../Actors%20states/IsStopped.md)</li></ul>|
|5|N/A|Move 2 if `data[0]` (amount of times revive was done) is at least 2 (at least 1 instead if hardmode is true) and a 51% RNG check passes|

1: There is an issue if move 3 or 4 (singing or relay) redirects to move 2 (kick) because `nonphysical` will remain true even if the kick move was not meant to be `nonphysical`. This will incorrectly affect the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target of the kick.

## Move 1 - Music note throw
A single target that may hit multiple times or party wide music note throw.

This move has 2 versions: the single target multi hit version and the party wide version. The single target version has 3/4 chance to happen while the party wide one has 1/4 chance to happen.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|1/4 chance to happen and if it does, it is done for each enemy party member whose `hp` is above 0|This enemy|The matching player party member|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|3/4 chance to happen and if it does, done from 1 to 2 times (from 1 to 3 times instead if hardmode is true)|1|[Pierce](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new GameObject named with a SpriteRenderer using a random sprite in `Sprites/Particles/music` (music note sprites) using a color of a RainbowColor from 0 to 9 variants and a SpinAround with an `itself` of (0.0, 0.0, 2.0)|35 (20 instead if hardmode is true)|0.0|null|null|`PingShot`|null|Vector3.zero|false|

### Logic sequence

- `Prefabs/Particles/MusicSimple` instantiated near this enemy
- animstate set to 103
- All `Sprites/Particles/music` preloaded

From there, this is where the version of the move is used. See above for the odds of which one gets used.

#### Single target multiple hits

- `StatUp2` sound plays
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- The amount of hits is determined which is random between 1 and 4 times (between 2 and 4 times instead if hardmode is true)
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- Yield for 0.5 seconds
- For each hit to do:
    - `PingUp` sound plays
    - A new GameObject with a SpriteRenderer is created using a random sprite from `Sprites/Particles/music` using a color of RainbowColor from 0 to 9 variants (determined at random) and a SpinAround with an `itself` of (0.0, 0.0, 2.0). The object is positioned at the this enemy + (0.0, 2.0, 0.1)
    - Over the course of 20.0 frames, the music note moves to 3.0 in y and 0.1 in z with a lerp and has its SpinAround's `itself`'s z lerped to 15.0. For the new x position, it's positioned such that it is equidistant horizontally at the entity's x position -2.0 
- `Prefabs/Particles/MusicSimple` positioned offscreen and destroyed in 2.0 seconds
- animstate set to 101
- Yield for 0.5 seconds
- For each hit to do
    - `Ping` sound plays with a pitch of 1.0 + the hit index starting from 0 * 0.1
    - Projectile 1 call happens
    - Yield for 0.7 seconds
- Yield all frames until all projectiles fired are destroyed (All Projectile call completed)
- Yield for 0.5 seconds

#### Party wide throw

- `Charge4` sound plays
- Camera moves to look near this enemy
- Yield for 0.5 seconds
- A new GameObject with a SpriteRenderer is created using a random sprite from `Sprites/Particles/music` using a color of RainbowColor from 0 to 9 variants (determined at random) and a SpinAround with an `itself` of (0.0, 0.0, 2.0). The object is positioned at the this enemy + (0.0, 2.0, 0.1)
- Over the course of 40.0 frames, the music note moves +2.5 in y with a lerp, gets scaled to 3x with a lerp and has its SpinAround's `itself`'s z lerped to 20.0
- animstate set to 101
- `Prefabs/Particles/MusicSimple` positioned offscreen and destroyed in 2.0 seconds
- Yield for 0.5 seconds
- Camera moves to look near the player party
- `Launch` sound plays
- Over the course of 50 frames (40 frames instead if hardmode is true), the music note moves to `partymiddle` using a BeizierCurve3 with a ymax of 9.0
- `musicexplosion` particles played at `partymiddle`
- `impactsmoke` particles played at `partymiddle`
- `BigHit` sound plays
- ShakeScreen for (0.15, 0.15, 0.15) ammount and 0.25 time
- For each player party members whose `hp` is above 0, DoDamage 1 call happens
- The music note object is destroyed
- Yield for 0.5 seconds

## Move 2 - Kick attack
A single target kick attack that may hit multiple times

This may redirect to move 1 (music note throw), check the move selection section above for details.

### About `nonphyscal` incorrectly being set to true
If this move is performed as a result from a redirection of move 3 or 4 (singing or relay), `nonphysical` will be set to true, but this move was never supposed to do this meaning this is undesired. This will incorrectly affect the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null if `commandsuccess` is false and {[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim)} if it's true|`commandsuccess`|
|2|Only happens if `commansuccess` was false in DoDamage 1 and if it was, done 3 times. NOTE: This doesn't take into account FRAMEONE meaning a regular block will prevent this call from happening even if DoDamage 1 ignored it due to FRAMEONE|null|The same as DoDamage 1|1|null|null|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentity`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (0.6, 0.0, -0.15) with a 1.5 multiplier with 100 walkstate and 104 stop state
- Yield for a frame
- Yield all frames until `forcemove` is done
- Camera stops targetting anyone
- Yield for 0.1 seconds
- `MothivaKick` sound plays
- DoDamage 1 call happens
- If `commandsuccess` is false (not blocked):
    - Yield for a frame
    - `Turn` sound plays
    - Camera moves slighly down and forward
    - `playertargetentity` animstate set to 11 (`Hurt`)
    - `playertargetentity` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0
    - `playertargetentity` y `spin` set to -15.0
    - Yield for 0.5 seconds
    - Yield all frames until `playertargetentity` is `onground`
    - Yield for a frame
    - `Death3` sound plays
    - `playertargetentity`'s `spin` zeroed out
    - `playertargetentity` has FlipAngle called on them with setangle
    - `playertargetentity` animstate set to 18 (`KO`)
    - DeathSmoke particles plays at `playertargetentity`
    - ShakeScreen for 0.1 time
    - Yield for 0.5 seconds
    - Done 3 times:
        - animstate set to 106
        - Yield for 0.15 seconds
        - animstate set to 107
        - DoDamage 2 call happens
        - `playertargetentity` has ShakeSprite called on them for (0.1, 0.0, 0.0) intensity and 0.1 frametimer
        - Camera moves slightly down and forward
        - Yield for 0.235 seconds
    - Yield for 0.75 seconds

## Move 3 - Singing
Sings which inflicts the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to [Zasp](Zasp.md). No damages are dealt.

This may redirect to move 1 or 2 (music note throw or kick), check the move selection section above for details.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

NOTE: It also incorrectly affects move 2 (kick) if redirected to it from this move.

### Logic sequence

- The `enemydata` index of [Zasp](Zasp.md) is obtained
- animstate set to 103
- `Prefabs/Particles/MusicSimple` instantiated and positioned near this enemy
- `StatUp2` sound plays
- Yield for 1 second
- `Zasp` animstate set to 106
- `BugWingFast` sound plays
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on `Zasp` to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 2 turns (effectively 1 turn since the main turn gets advanced soon after)
- [StatEffect](../../Visual%20rendering/StatEffect.md) on `Zasp` with type 0 (blue up arrow)
- Yield for 0.65 seconds
- `Prefabs/Particles/MusicSimple` positioned offscreen and destroyed in 2.0 seconds

## Move 4 - Relay
Relay her turn which gives an actor turn to [Zasp](Zasp.md). No damages are dealt.

This may redirect to move 1 or 2 (music note throw or kick), check the move selection section above for details.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

NOTE: It also incorrectly affects move 2 (kick) if redirected to it from this move.

### Logic sequence

- The `enemydata` index of [Zasp](Zasp.md) is obtained
- animstate set to 4 (`ItemGet`)
- `Zasp` animstate set to 2 (`Jump`)
- y `spin` set to 15.0
- `Zasp`'s y `spin` set to 15.0
- `Zasp`'s `cantmove` decremented which gives them an extra actor turn
- `Relay` sound plays
- Yield for 0.5 seconds
- `spin` zeroed out
- `Zasp`'s `spin` zeroed out
- `Zasp` animstate set to its `basestate`
- animstate set to the `basestate`

## Move 5 - Revive
Revives [Zasp](Zasp.md) from the `reservedata`

This may redirect to move 2 (kick), check the move selection section above for details.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any targets.

Unlike move 3 and 4 (singing and relay), it won't incorrectly affect move 2 (kick) if redirected to it from this move.

### Logic sequence

- If `data[0]` is 0 (no revives was done before):
    - [SetText](../../../SetText/SetText.md) called in [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[95]`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - cameraoffset: Vector3.zero
        - size: Vector3.one
        - parent: this enemy
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
    - Yield for 0.5 seconds
- `data[0]` incremented
- `MothivaRes` sound plays
- animstate set to 103
- `Prefabs/Particles/MusicSimple` instantiated and positioned near this enemy
- Over the course of 40.0 frames, all Light under the `battlemap` has their intensity lerped to be halved
- [ReviveEnemy](../ReviveEnemy.md) called with [Zasp](Zasp.md)'s `reservedata` index with an `hp` of 20% of the `maxhp` ceiled without canmove
- The `enemydata` index of `Zasp` is obtained
- `Prefabs/Objects/Spotlight` instantiated near `Zasp`
- Yield for 1 second
- `Heal` sound plays
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP counter) with an ammount of `Zasp`'s `hp` starting at (0.0, 2.0, 0.0) and ending at (0.0, 1.0, 0.0)
- Yield for 0.6 seconds
- `Zasp` animstate set to 105
- Yield for a second
- `Zasp` animstate set to 13 (`BattleIdle`)
- `Prefabs/Objects/Spotlight` destroyed
- `Prefabs/Particles/MusicSimple` positioned offscreen and destroyed in 2.0 seconds
- Over the course of 40.0 frames, all Light under the `battlemap` has their intensity lerped to be their value before this action
- All Light under the `battlemap` has their intensity set to be their value before this action
