# `AngryPlant`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: The amount of actor turns left `Flying` before dropping to `Ground`. This is set to 3 during the `hitaction` logic and decremented every actor turns's action. When it reaches 0 (or 1 if this enemy is the only remaining enemy party member), this enemy will drop down to `Ground` which will cause the value to be set to -1.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 2 changes:

- The healing move on another party member has an RNG check of 71% instead of 51% if all the other conditions applies
- The vine attack, wavy projectile and delayed projectile moves have different odds distribution
- The vine attack move may hit twice instead of only once
- In the wavy projectile move, the projectile takes 50.0 frames to reach its target instead of 65.0 frames
- In the multiple projectiles throw move, the projectile takes 1 / 0.03 frames (~33.333334 frames) for the projectile to move to its target instead of 1 / 0.025 frames (40 frames). 
- In the multiple projectiles throw move, the yield time between projectiles changes to 0.225 seconds instead of 0.3 seconds

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves

### `nonphyscal` set to true
This move always sets `nonphyscal` to true. Since no damages are dealt in the `hitaction` logic, this doesn't do anything.

### Logic sequence
The overall logic causes a [position](../../Actors%20states/BattlePosition.md) set to `Flying` with `data[0]` set to 3.

- `data[0]` set to 3
- animstate set to 101
- `emoticonid` set to 2 (red ! mark) with an `emoticoncooldown` set to 30.0
- `Wam` sound plays
- Yield all frames until `emoticocooldown` expires
- y `spin` set to -20.0
- `Rumble3` sound plays
- ShakeScreen called for 0.7 time
- `DirtExplode` particles plays at this enemy position
- y `spin` set to -30.0
- `spinextra[0]` set to (0.0, -20.0, 0.0)
- `Charge6` sound plays
- Over the course of 40.0 frames, `height` changes to the y component of a BeizierCurve from the current postion to the same + 3.0 in y with a ymax of 5.0. Before each frame yield, `oldstate` is set to -1
- `bobrange` and `bobspeed` set to `startbf` and `startbs` respectively
- `spin` zeroed out
- Yield for 0.5 seconds
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`

## Move selection
4 moves are possible:

1. A single target vine attack that may hit multiple times
2. A single wavy projectile throw
3. Adds a single target [DelayedProjectile](../../Actors%20states/Delayed%20projectile.md)
4. A heal breath that heals the `hp` of another enemy party member
5. A single target multiple projectiles throw

Move 5 can only be used when [position](../../Actors%20states/BattlePosition.md) is `Flying` and it is always used when this happens.

Move 4 will only be used under the following conditions:

- [position](../../Actors%20states/BattlePosition.md) is `Ground`
- A 51% RNG check passes (71% instead if hardmode is true)
- An enemy party member exists other than this enemy whose `hp` / their `maxhp` floored is less than 0.6 (less than 60% `hp` remaining) and has a [position](../../Actors%20states/BattlePosition.md) of `Ground`

If the above are fufilled, the first enemy party member found will be targetted by the move and move 4 will always be used under these conditions.

If move 4 or 5 aren't used, the odds of using move 1-3 depends on hardmode being true or not. Here are the odds:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|---------------------------|-------------------------|
|1|3/6|2/7|
|2|2/6|3/7|
|3|1/6|2/7|

## Pre move logic
The following logic happens before any move is used:

- If `data[0]` is above 0, it is decremented
- If `data[0]` is now 0 (or 1 if it's the only enemy party member left) while [position](../../Actors%20states/BattlePosition.md) is `Flying`, it drops to `Ground` with the following logic:
    - `data[0]` set to -1
    - `ChargeDown` sound plays
    - animstate set to 101
    - y `spin` set to -20.0
    - Over the course of 30.0 frames, `height` is lerped to 0.0
    - `height` set to 0.0
    - animstate set to 0 (`Idle`)
    - `oldfly` toggled
    - `spin` zeroed out
    - FlipAngle called with setangle
    - [position](../../Actors%20states/BattlePosition.md) set to `Ground`
    - Yield for 0.25 seconds

## Post move logic
If any move except move 5 were used (basically, anything that requires a [position](../../Actors%20states/BattlePosition.md) of `Ground`), the following logic happens after the move:

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
- [Heal](../../Actors%20states/Heal.md) this enemy for 1 `hp`
- Yield for 0.5 seconds

## Move 1 - Vine attack
A single target vine attack that may hit multiple times

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|`playerdata[playertargetID]`'s `hp` is above 0. Done once if hardmode is false, from 1 to 2 times if it is true as long as `playerdata[playertargetID]`'s `hp` is above 0|This enemy|The selected `playertargetID`|2 on the first hit, 1 on the second hit|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.5 seconds
- `checkingdead` is set to a new VineAttack coroutine with 2 damageamt, actionid as the callerid without gettarget. The coroutine will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` goes to null

This is what the coroutine effectively ends up doing:

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

## Move 2 - Wavy projectile
A single wavy projectile throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|[Sleep](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 103
- Yield for 0.1 seconds
- A new GameObject is created rooted with a SpriteRenderer using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at this enemy + (-0.9, 1.0, -0.1) with a ShadowLite that is SetUp with 0.4 opacity and 0.5 size
- Yield a frame
- `Spit` sound plays
- `WavyLoop` sound plays on loop with 0.75 volume
- Over the course of 65.0 frames (50.0 frames instead if hardmode is true), the projectile object is lerped to `playertargetentity` position + (0.0, 1.0, 0.0) which is then added to (0.0, Sin(Time.time * 15.0) * 0.3, 0.0) which gives a vertical oscillation travel. Before each frame yield, the z angle increases by 10x the game's frametime
- `WavyLoop` sound stopped
- DoDamage 1 call happens
- Yield a frame
- The projectile object is destroyed
- Yield for 0.5 seconds

## Move 3 - Delayed projectile
Adds a single target [DelayedProjectile](../../Actors%20states/Delayed%20projectile.md). No damages are dealt on this actor turn.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true, but it doesn't affect anything for this move.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Always happen|A new sprite object using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at (0.0, 10.0, 0.0) childed to the `battlemap` with a SpinAround that has an `itself` of -10.0 in z|A random `plauyerdata` index|2|2|-1|[Sleep](../../Damage%20pipeline/AttackProperty.md)|60.0|This enemy|null|null|`@ChargeDown3`|

### Logic sequence

- Camera moves to look near this enemy
- animstate set to 100
- Yield for 0.75 seconds
- animstate set to 105
- Yield for 0.15 seconds
- animstate set to 106
- Yield for 0.15 seconds
- `Spit` sound plays
- `Charge5` sound plays
- A new sprite object is created using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at (0.0, 10.0, 0.0) childed to the `battlemap` with a SpinAround that has an `itself` of -10.0 in z|
- Over the course of 50.0 frames, the projectile object moves to (this enemy - 1.0, 10.0, 0.0) with a lerp and scales up to 1.5x with a lerp, but that only grows over the course of the first 30.0 frames
- Yield a frame
- The projectile object position is set to (0.0, 10.0, 0.0)
- AddDelayedProjectile 1 call happens
- Yield for 0.3 seconds

## Move 4 - Heal breath
A heal breath that heals the `hp` of another enemy party member. No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true, but it doesn't affect anything for this move.

### Logic sequence

- Camera moves to look near the midpoint between this enemy and the target enemy party member
- [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) the target enemy party member
- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 103
- Yield for 0.1 seconds
- `HealBreath` sound plays
- `HealSmoke` particles played near this enemy with an alivetime of 5.0
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called on the target enemy party member to heal an amount of `hp` equal to its `maxhp` * 0.1 clamped from 3 to 99
- Yield for 1.15 seconds

## Move 5 - Multiple projectiles
A single target multiple projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 to 3 times|1|[Sleep](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new GameObject rooted with a SpriteRenderer using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at this enemy + (0.9, 1.0 + `height`, -0.1) with a ShadowLite that is SetUp with 0.4 opacity and 0.5 size|0.025 (40.0 frames of movement) if hardmode is false, 0.03 if it's true (~33.333334 frames of movement)|0.0|null|null|null|null|(0.0, 0.0, 20.0)|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 103
- Yield for 0.1 seconds
- The amount of hits is determined to be random between 2 and 3 and it is used to initialise the array of projectiles objects
- Done for each hit:
    - `Spit` sound plays
    - A new GameObject is created rooted with a SpriteRenderer using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at this enemy + (0.9, 1.0 + `height`, -0.1) with a ShadowLite that is SetUp with 0.4 opacity and 0.5 size. The projectile is assigned to the corresponding element of the projectiles array
    - Yield for a frame
    - Projectile 1 call happens using the GameObject mentioned above
    - Yield for 0.3 seconds (0.225 seconds instead if hardmode is true)
- animstate set to 0 (`Idle`)
- Yield all frames until all elements of the projectiles array are null (meaning they were all destroyed so all Projectile calls completed)
- Yield for 0.2 seconds
