# `ZombieRoach`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 4 element with a starting value of 0 for each element except `data[1]` where the starting value is 2.

- `data[0]`: This tells the "mode" this enemy is on which drastically changes the move selection process because it changes the sets of move they can use. The value can only be 0 or 1 and it is toggled between the 2 when `data[1]` is 0 at the end of the action which will cause them to switch the current mode in the post move logic and have `data[1]` set back to 2 (meaning the current mode won't change until 2 main turns). Here are the details of the 2 modes:
    - 0: Sand mode. Here are the moveset changes:
        - The energy spheres throw move is accessible, but not the single target ice attack move
        - The moving sand attack move is accessible, but not the party wide icicle throw move
        - When summoning an enemy, it will always be a [SandWall](SandWall.md)
        - After using a magic boost or heal, the energy spheres throw move will always be used
    - 1: Ice mode. Here are the moveset changes:
        - The single target ice attack move is accessible, but not the energy spheres throw move 
        - The party wide icicle throw move is accessible, but not the moving sand attack move
        - When summoning an enemy, it will always be an [IceWall](IceWall.md)
        - After using a magic boost or heal, the single target ice attack move will always be used
- `data[1]`: This is the main turn cooldown before `data[0]` is toggled which causes a mode switch. Its starting value is 2 and it decreases at the end of their last actor turn of the main turn in the post move logic. When it reaches 0 after the decrement, `data[0]` is toggled between 0 and 1 which causes a mode change
- `data[2]`: This is an actor turn cooldown that restricts the usage of the moving sand move and party wide icicle throw move. When the former is used, the value is set to 2 and when the latter is used, the value is set to 3. At the start of every actor turn in the pre move logic, the value decrements when it is above 0. For either moves to be selectable (whichever one is appropriate for the current mode), the value needs to be 0 after the decrement. However, this doesn't apply if this enemy's [position](../../Actors%20states/BattlePosition.md) is `Underground` as the moving sand move is always used in that case. It also doesn't completely prevent either move to be used if the enemy selects the wall enemy summon move while there is already one in `enemydata`. See the move selection section below for details
- `data[3]`: Effectively UNUSED because while it is written to when this enemy heals themselves and it is decremented in the post move logic if it's higher than 0, it is never read meaning this value effectively doesn't do anything

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The [HPPercent](../../Actors%20states/HPPercent.md) threshold changes for getting a [moves](../../Actors%20states/Enemy%20features.md#moves) of 2 and a decremented `cantmove` (2 actor turns per main turns including the current one). It changes to be when it reaches lower than 0.5 from 0.33
- In the energy spheres throw move, the amount of projectiles to throw changes to be random between 2 and 3 inclusive instead of always being 2
- In the energy spheres throw move, the base damage dealt per each projectile changes to 2 from 3
- In the energy spheres throw move, the yield time before each Projectile calls changes to 0.5 seconds from 0.65 seconds
- In the energy spheres throw move, the speed of each projectile changes to 40.0 from 50.0
- In the single target ice attack move, the amount of frames the ice pillar takes to reach its target and appear changes to 71.0 frames from 81.0 frames
- In the moving sand attack move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the moving sand attack move, the damageammount of each DoDamage calls changes to 0 from 1 (this doesn't mean the call no longer does damages because it is meant for the `hardatk` effect to apply which could deal a non zero amount of damages and the call also has [AtLeast1Pierce](../../Damage%20pipeline/AttackProperty.md) which guarantees a non zero amount of damages)
- In the enemy summon move, `cantmove` gets decremented which grants an additonal actor turn to this enemy

## Move selection
6 moves are possible:

1. A multiple targets energy spheres throw
2. A single target ice attack
3. A single target moving sand attack that may hit multiple times and drain `hp`
4. A party wide icicle throw
5. Summons a new [SandWall](SandWall.md) or [IceWall](IceWall.md) enemy
6. Uses magic to boost or heal themselves, then performs move 1 or 2

The move selection process is complex because it involves the current mode (see the `data[0]` explanation above), odds (which can change depending on `data[2]`) as well as some special cases.

For the mode, it changes which pair of moves is usable amongst move 1 through 4 and it also decides the enemy to summon in move 5. The other pair is innacessible and will never be used until the mode toggles. Here is a table that summarises each mode (the starting mode is the sand mode):

|Mode|Moves accessible|Enemy summoned on move 5|
|---:|----------------|------------------------|
|Sand|<ul><li>1 - Energy spheres throw</li><li>3 - Moving sand attack</li></ul>|[SandWall](SandWall.md)|
|Ice|<ul><li>2 - Single target ice attack</li><li>4 - Party wide icicle throw</li></ul>|[IceWall](IceWall.md)|

What's important is that move 1 and 2 as well as move 3 and 4 are viewed as the same during the move selection process. The actual move to use from each of these pairs is determined by the mode, but they still feature completely different logic which is why they are also documented as separate moves. For the sake of simplicity, each pair will be refered to as "move 1-2" and "move 3-4" since they are viewed the same way during the selection process.

Next, the odds to use a move. Here are the base odds to use each move:

|Move or pair|Odds|
|-----------:|----|
|1-2|2/7|
|3-4|2/7|
|5|1/7|
|6|2/7|

However, there are 2 special cases to this:

- If [position](../../Actors%20states/BattlePosition.md) is `Underground`, move 3-4 is always used (this should only and always be the case after using move 3 and in such case, move 3 will be used again)
- If move 3-4 was selected and `data[2]` is above 0 (after the decrement), the move can't be selected because the actor turn cooldown on it hasn't expired yet

In that second case, the entire move selection is rerolled with different odds that don't include move 3/4. Here are those special odds:

|Move or pair|Odds|
|-----------:|----|
|1-2|2/5|
|5|1/5|
|6|2/5|

Finally, move 5 may cause another move to be used instead once selected if there are more than 1 `enemydata` including this enemy (which means move 5 was used and the summoned enemy is still present). When this happens, move 1-2 or move 3-4 will be used which is determined by an RNG check with uniform probabilities (1/2 chance each).

## Pre move logic
The following logic always happen at the start of the action:

- If `data[2]` is above 0, it is decremented (this is the move 3-4 cooldown tracking)

Next is a phase transition that happens if all of the following conditions are fufilled:

- [BattlePosition](../../Actors%20states/BattlePosition.md) isn't `Underground`
- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.33 (0.5 instead if hardmode is true)
- This enemy's [moves](../../Actors%20states/Enemy%20features.md#moves) is 1 (meaning this transition didn't happen yet)

Here is what happens in the phase transition:

- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- animstate set to 100
- y `spin` set to 15.0
- Yield for 0.75 seconds
- `spin` zeroed out
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
- `moves` set to 2 and `cantmove` decremented (gives 2 actor turns per main turn including the current main turn)
- Yield for 0.5 seconds

## Post move logic
The following logic always happen after using a move:

- If `cantmove` is 0 (meaning this is the end of the last actor turn of this enemy which acts as a pseudo main turn ending check), `data[1]` is decremented. If after this decrement, it became 0 or below (meaning the main turn cooldown for toggling the mode just expired):
    - If [position](../../Actors%20states/BattlePosition.md) is `Underground`, it is changed to `Ground` with the following logic (it is assumed that the moving sand attack move was used):
        - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with a 2.0 multiplier
        - Yield all frames until `forcemove` is done
        - `digging` set to false
        - `flip` set to false
        - FlipAngle called with setangle
        - `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call to change this enemy's `position` to `Ground`
        - Over the course of 31.0 frames, `extra[1]` (a `Prefabs/Objects/MovingSand` from the moving sand attack move) scale changes from 0.5x to 0.0x via a lerp
        - `extra[1]` gets destroyed
        - Yield all frames until `checkingdead` is null (the coroutine completed)
    - `initialheight` set to 0.3
    - `data[0]` toggled (becomes 1 if it was 0 or becomes 0 if it was 1)
    - y `spin` set to 15.0
    - `Cloth` sound plays
    - If `data[0]` is 1 (ice mode):
        - The local startstate is set to 13 (`BattleIdle`)
        - `minheight` and `height` set to 0.0
        - animstate set to 102
    - Otherwise (sand mode):
        - The local startstate is set to 0 (`Idle`)
        - `height` and `minheight` set to `initialheight`
        - animstate set to 103
        - `bobspeed` and `bobrange` set to `startbs` and `startbf` respectively
    - `data[1]` set to 2 (reset the main turn cooldown before toggling modes again)
    - Yield for 0.5 seconds
    - `spin` zeroed out
    - FlipAngle called with setangle
    - `basestate` and animstate set to the local startstate
- If `data[3]` is above 0, it is decremented (this doesn't do anything since this `data` slot is effectively UNUSED)

## Move 1 - Energy spheres throw
A multiple targets energy spheres throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 times (random between 2 and 3 times instead if hardmode is true), but each call requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3 (2 instead if hardmode is true)|null|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each call)|A new `Prefabs/Objects/EnergySphere` GameObject rooted positioned at this enemy position + (-2.0, 4.0, 0.0) + a random vector between (-1.5, -1.0, 0.0) and (1.5, 1.0, 0.0) with a `NoMapColor` tag|50.0 (40.0 instead if hardmode is true)|1.0 (-0.5 instead if it's the second hit)|`SepPart@3@1,keepcolor`|`Stars`|`PingShot`|null|Vector3.zero|false|

### Logic sequence

- animstate set to 100
- Yield for 0.5 seconds
- The amount of hits is determined which is 2 (random between 2 and 3 instead if hardmode is true)
- `checkingdead` set to an EnergySpheres coroutine with the amount of hits to do (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null

This is what the EnergySpheres coroutine ends up doing:

- A GameObject array is created to hold each projectile objects
- For each hits to do:
    - If at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)
        - `Charge7` sound plays
        - A new `Prefabs/Particles/Charging` GameObject is created rooted positioned at this enemy position +  (-2.0, 4.0, 0.0) + a random vector between (-1.5, -1.0, 0.0) and (1.5, 1.0, 0.0)
        - Yield for 0.65 seconds (0.5 seconds instead if hardmode is true)
        - The projectile element is set to a new `Prefabs/Objects/EnergySphere` GameObject rooted positioned at the `Prefabs/Particles/Charging` position with a `NoMapColor` tag
        - `Lazer2` sound plays
        - Projectile 1 call happens
        - `Prefabs/Particles/Charging` moves offscreen at 999.0 in y
        - `Prefabs/Particles/Charging` gets destroyed in 3.0 seconds
        - Yield all frames until the projectile element is null (the Projectile call completed)
- Yield for 0.5 seconds
- `checkingdead` set to null which signals the caller that the coroutine completed

## Move 2 - Single target ice attack
A single target ice attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Freeze](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)
- animstate set to 101
- `IceMothThrow` sound plays
- A new `Prefabs/Objects/SingleSphere` GameObject is created childed to a new `Prefabs/AnimSpecific/mothbattlesphere` GameObject
- Over the course of 31.0 frames, the `Prefabs/AnimSpecific/mothbattlesphere` moves from this enemy position + (1.1, 1.85, -0.1) to this enemy position + (-2.0, -0.5, 5.0) via a BeizierCurve3 with a ymax of 5.0
- The `Prefabs/Objects/SingleSphere` gets rooted
- If `hologram` is true, the `Prefabs/Objects/SingleSphere` Renderer's color is set to 8080FF (mostly blue) with half opacity
- `Prefabs/AnimSpecific/mothbattlesphere` gets destroyed
- `WatcherIce` sound plays
- Over the course of 81.0 frames (71.0 instead if hardmode is true), `Prefabs/Objects/SingleSphere` moves from its position with the y component at 0.0 to `playertargetentity` position via a SmoothLerp
- `Prefabs/Objects/SingleSphere` gets destroyed
- animstate set to 104
- `IceMothHit` sound plays
- A new `Prefabs/Particles/mothicenormal` GameObject is created rooted at `playertargetentity` position + 0.5 in y then destroyed in 2.0 seconds
- A new `Prefabs/Objects/icepillar` is created rooted at `playertargetentity` position with a DialogueAnim that has a `targetscale` of (0.5, 0.5, 1.0)
- DoDamage 1 call happens
- Yield for 0.65 seconds
- The `Prefabs/Objects/icepillar`'s DialogueAnim has its `shrink` set to true with a `shrinkspeed` of 0.025
- The `Prefabs/Objects/icepillar` gets destroyed in 5.0 seconds
- Yield for 0.5 seconds

## Move 3 - Moving sand attack
A single target moving sand attack that may hit multiple times and drain `hp`.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|-1.0 (infinite)|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Called once if `barfill` is less than 1.0 (not full) during DoCommand 1 once 0.5 seconds passed and then continously called again every 1.25 seconds until either `barfill` reaches 1.0 (checked 0.75 seconds after each call) or that `playerdata[playertargetID]`'s `hp` becomes 0|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target is the same for each calls)|1 (0 instead if hardmode is true<sup>1</sup>)|[Atleast1pierce](../../Damage%20pipeline/AttackProperty.md)<sup>2</sup>|null|false|

1: This doesn't mean the call won't deal any damage because not only `Atleast1pierce` guarantees 1 damage is dealt, but also because it is meant to function with `hardatk` in mind which takes effect when hardmode is true
2: Enemy piercing damages are disabled, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- `data[2]` set to 3 (this is the actor turn cooldown to use this move or the party wide icicle throw, it will effectively be 2 actor turns since the decrement happens before)
- `WatcherSandLoop` sound plays on loop using `sounds[9]`
- If this enemy [position](../../Actors%20states/BattlePosition.md) isn't already `Underground`, it is changed to it with the following:
    - `digging` set to true
    - `checkingdead` set to a new [ChangePosition](../ChangePosition.md) call to change this enemy's `position` to `Underground`
    - `extra[1]` initialised to a new `Prefabs/Objects/MovingSand` GameObject childed to this enemy with a Renderer's color of pure gray
    - Over the course of 61.0 frames, `extra[1]` scale changes from 0.0x to 0.5x via a lerp
    - Yield all frames until `checkingdead` is null (the coroutine completed)
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity`
- Yield all frames until `forcemove` is done
- Camera moves to look near `playerdata[playertargetID]`
- y `spin` set to 13.0
- `playertargetentity` animstate set to 11 (`Hurt`)
- Over the course of 30.0 frames, `playertargetentity` moves to -0.5 in y via a lerp
- `infinitecommande` set to true which will prevent DoCommand 1 to end when MainManager.`mashcommandalt` is true until the action command is succeeded
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the [TappingKey](../../Action%20commands/TappingKey.md) command's help)
- DoCommand 1 call happens
- As long as `playerdata[playertargetID]`'s `hp` is above 0 and `actionroutine` isn't null (meaning DoCommand 1 hasn't ended yet):
    - Yield for 0.5 seconds
    - If `barfill` is less than 1.0 (it isn't full yet):
        - DoDamage 1 call happens
        - If `lastdamage` is above 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy by `lastdamage` amount of `hp`
    - Yield for 0.75 seconds
- `Barfill` sound stops
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- FinishAction is called manually to end DoCommand 1 if it wasn't already (but it should be by then)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) + 3.0 in x
- `playertargetentity`'s y `spin` set to 20.0
- Over the course of 30.0 frames, `playertargetentity` moves from its position - 0.5 to its position before this action via a BeizierCurve3 with a ymax of 2.0
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `playertargetentity`'s `spin` zeroed out
- `sounds[9]` stopped
- If `playertargetentity`'s `hp` is above 0, its animstate is set to the value they had before this action
- Yield all frames until `forcemove` is done

## Move 4 - Party wide icicle throw
A party wide icicle throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Freeze](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 100
- A new `Prefabs/Objects/icecle` GameObject is created rooted positioned at this enemy position + (-2.0, 4.5, 0.0) with a scale of 0.0x and its BoxCollider destroyed
- `Spin` sound plays on loop using `sounds[8]`
- Over the course of 60.0 frames, `Prefabs/Objects/icecle` scale changes to 1.25x via a lerp, the y angle goes towards 10.0 and the `sounds[8]` pitch changes from 0.0 to 1.0
- A new UI object is created named `t` childed to the `GUICamera` using the `guisprites[41]` sprite (a crosshair) on layer 15 (`3DUI`)
- animstate set to 101
- `Crosshair` sound plays on loop using `sounds[9]` with 0.9 pitch and 0.35 volume
- Over the course of 81.0 frames, `t` moves from (0.0, 3.0, 0.0) to (-4.5, 1.0, 0.0) via a SmoothLerp + a number in the y component that goes from Sin(Time.time * 2.0) * -3.0 to 0.0. Before each yield, `t` z angle increases by 5x the game's frametime and `Prefabs/Objects/icecle` y angles increases by 10.0
- `sounds[8]` and `sounds[9]` stopped
- `IceMothThrow` sound plays
- `t` gets destroyed
- `Prefabs/Objects/icecle` angles set to (0.0, 0.0, -45.0)
- Over the course of 26.0 frames, `Prefabs/Objects/icecle` moves to (-4.5, 1.0, 0.0) via a lerp
- `IceMothHit` sound plays
- `mothicenormal` particles plays at the `Prefabs/Objects/icecle` position with a scale of 2.0x
- `Prefabs/Objects/icecle` gets destroyed
- PartyDamage 1 call happens
- Yield for 0.5 seconds

## Move 5 - Wall enemy summon
Summons a new [SandWall](SandWall.md) or [IceWall](IceWall.md) enemy. No damages are dealt.

### Logic sequence

- `Charge12` sound plays
- animstate set to 100
- Yield for 0.5 seconds
- ShakeScreen called with 0.2 ammount and 0.75 time
- Yield for 0.75 seconds
- animstate set to 104
- `Charge15` sound plays
- `impactsmoke` particles plays at this enemy position - 3.0 in x
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [SandWall](SandWall.md) enemy (in sand mode) or [IceWall](IceWall.md) enemy (in ice mode) at this enemy position - 3.0 in x
- Yield all frames until `checkingdead` is null (the coroutine completed)
- If hardmode is true, `cantmove` is decremented which grants an additional actor turn to this enemy
- `enemydata[1]` (which should always be the wall that was just summoned) has its `diebyitself` set to true which mean if they are the only enemy party member alive, [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md) will kill them

## Move 6 - Magic boost or heal followed by move 1 or 2
Uses magic to boost or heal themselves, then performs move 1 (Energy spheres / Single target ice). No damages are dealt for this move, but move 1 will causes damages to be dealt.

### Logic sequence

- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- animstate set to 100
- y `spin` set to 15.0
- Yield for 0.75 seconds
- `spin` zeroed out

From there, 3 effects can occur: a heal, [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. The one that will be done depends on 2 checks:

1. `locktri` is false and a 2.5/10.0 RNG check passes
2. [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.4 and a 5/10 RNG check passes

Here is a table that summarises which one will be done when according to these 2 checks (note that check 2 is only tested if check 1 was true):

|Check 1|Check 2|Effect|
|-------|-------|------|
|false|N/A|[DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md)|
|true|false|[AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|
|true|true|Heal|

The `locktri` requirement for the `AttackUp` and heal is set to true when the heal effect happens. This means that healing will prevent this enemy from ever healing or getting the `AttackUp` condition again by the usage of this move.

Here is what happens in the `DefenseUp` condition effect:

- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 1 (blue up arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy for 2 main turns

Here is what happens for the `AttackUp` condition effect:

- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 0 (red up arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 2 main turns
- `StatUp` sound plays

And here's what happens in the healing effect:

- [Heal](../../Actors%20states/Heal.md) is called to heal this enemy by its `maxhp` * 0.1 floored
- `locktri` is set to true (this will prevent this enemy to heal or get an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition by using this move again)
- `data[3]` set to 3 (this doesn't do anything because this `data` slot is UNUSED)

Finally, after the effect is done, the following happens:

- Yield for 0.5 seconds
- Move 1 or 2 (whichever is appropriate based on the current mode) is used immediately without ending this actor turn
