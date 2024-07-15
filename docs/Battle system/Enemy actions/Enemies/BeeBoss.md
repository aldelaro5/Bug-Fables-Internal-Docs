# `BeeBoss`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 4 element with a starting value of 0.

- `data[0]`: The amount of actor turns left before being able to use the summon move again after it was just used. When the move is used, the value is set to 3 meaning this enemy cannot use the summon move for the next 3 actor turns (the missiles throw move will be used instead if selected). The value is decremented in the pre move logic
- `data[1]`: The amount of actor turns left before being able to launch a [delayed projectile](../../Actors%20states/Delayed%20projectile.md) in the post move logic again after one was just launched. When a delayed projectile is launched, the value is set to 4 meaning this enemy cannot launch another delayed projectile for the next 4 actor turns. The value is decremented in the pre move logic
- `data[2]`: This is set to 1 when the phase transition occurs in the pre move logiic when [HPPercent](../../Actors%20states/HPPercent.md) reaches lower than 0.6 so it prevents the transition to happen again
- `data[3]`: A counter that is set to 3 when the charge move is used and gets decremented if it was above 0 in the post move logic on every actor turns except when the charged dash attack move is used. The charge move requires this value to be at 0 to be used again which effectively means that when the charge move is used, it will not be used for the next 3 actor turns (the next one isn't counted and the next 2 won't have a charge usage because this value will be above 0). If the charge move is selected and this value is above 0, the missiles throw move will be used instead

## [StartBattle](../../StartBattle.md) special logic
This enemy has some special logic that happens on StartBattle after they are loaded:

- `battlepos` set to (4.5, 0.0, 0.0)
- position set to their `battlepos`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the missiles throw move, the amount of missiles thrown changes to 3 from 2
- In the missiles throw move, the speed of each projectile changes to 40.0 from 50.0
- In the missiles throw move, the yield time between each throw changes to 0.65 seconds from 0.85 seconds

## Move selection
5 moves are possible:

1. A party wide laser attack
2. A multiple targets missiles throw
3. Summon a new [BeeBot](BeeBot.md) enemy
4. Prepares an attack next turn which increments `charge`
5. A party wide charged dash attack

Move 5 will always be used (and can only be used) when `basestate` isn't 0 (`Idle`) or `charge` is above 0. NOTE: This is assumed to be the case after move 4 was performed last actor turn since this one ultimately changes the `basestate` away from 0 (`Idle`) meaning it is assumed that move 4 will only be used when move 3 was the last move performed. It doesn't seem to be possible under normal gameplay to get into a false positive.

Move 1 through 4 usage are based on odds, but the odds changes depending if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 or not. Additionally, some moves have additional requirements that needs to be fufilled upon selecting the move. If those requirements aren't fufilled when the move is selected, another move will be used instead.

|Move|Odds when [HPPercent](../../Actors%20states/HPPercent.md) >= 0.5|Odds when [HPPercent](../../Actors%20states/HPPercent.md) < 0.5|Requirements|Move used when requirements failed|
|---:|-------------------------------|-----------------------------|------------|---------------------------------|
|1|1/4|1/6|None|N/A|
|2|1/4|2/6|None|N/A|
|3|1/4|1/6|<ul><li>This is the last enemy party member left</li><li>`data[0]` is 0 (the actor turn cooldown on this move exhausted)</li><li>[HPPercent](../../Actors%20states/HPPercent.md) is above 0.2</li></ul>|Move 2|
|4|1/4|2/6|<ul><li>`data[3]` is 0 (the actor turn cooldown on this move exhausted)</li><li>[HPPercent](../../Actors%20states/HPPercent.md) is between 0.15 exclusive and 0.65 exclusive</li></ul>|Move 2|

## Pre move logic
There is logic that always happen at the start of the action. It's split in 3 parts that happens in the order mentioned.

### Change of [position](../../Actors%20states/BattlePosition.md) to `Flying`
The following logic always happen if [position](../../Actors%20states/BattlePosition.md) is `Ground`:

- ShakeSprite called with intensity (0.15, 0.0, 0.0) and 50.0 frametimer
- animstate set to 0 (`Idle`)
- `b33Shake` sound plays
- Yield for 0.8 seconds
- `b33BotRisingFromGround` sound plays
- Over the course of 51.0 frames, `height` changes to the y component of a BeizierCurve3 from Vector3.zero to (0.0, `initialheight`, 0.0) with a ymax of `initialheight` * 2
- `height` set to `initialheight`
- `bobrange` and `bobspeed` set to `startbf` and `startbs` respectively
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`

### `data` cooldowns decrement
The following logic always happen:

- If `data[0]` is higher than 0, it is decremented (this is the summon move cooldown)
- If `data[1]` is higher than 0, it is decremented (this is the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) cooldown)

### Phase transition
The following logic always happen if `data[2]` is 0 (this phase transition hasn't happened yet) and [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.65:

- animstate set to 108
- `B33BotMalfunction` sound plays
- Done 3 times:
    - ShakeSprite called with intensity (0.2, 0.2, 0.2) and 30.0 frametimer
    - DeathSmoke particles plays at this enemy position + (0.0, `height` + 1.5, -0.25) + a random vector between (-2.5, -0.75, 0.0) and (2.5, 0.75, 0.0)
    - `b33BotCracking` sound plays
    - Yield for 0.8 seconds if this isn't the last iteration
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 9999999 main turns
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition on this enemy for 9999999 main turns
- The sprite of the `model`'s first child (their main body part) is set to `Sprites/Entities/beeboss` index 12 (a cracked body party)
- The `model`'s 6th child (weak smoke particles) has its ParticleSystem played
- `data[2]` set to 1 (so this phase transition doesn't happen again)
- Yield for 0.85 seconds
- animstate set to `basestate`
- Yield for 0.3 seconds

## Post move logic
There is logic that always happen after a move is used with the exception of move 5 being used (the charged dash attack) where none of the logic happens. It notably features a [DelayedProjectile](../../Actors%20states/Delayed%20projectile.md) being launched.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|<ul><li>[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable didn't return -1</li><li>`data[1]` is 0 (the cooldown on this delayed projectile exhausted)</li><li>`basestate` is 0 (`Idle`) meaning the charged dash attack move wasn't used on this enemy's last actor turn</li></ul>|A new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at (0.0, 20.0, 0.0) with a z angle of 75.0|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|4|3|0|null|60.0|This enemy|`Explosion3`|`explosion`|`MisseleFall`|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- If `data[3]` is above 0, it is decremented (this is the cooldown for the charging move)
- [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable is called. If it returns -1, the action ends early
- Camera moves to look near this enemy, but a bit up
- animstate set to 105
- `B33BotHatchOpen` sound plays
- Yield for 0.75 seconds
- animstate set to 106
- A new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at `model`'s third child position (their front canon) + (-0.2, 0.8, -0.2) with a z angle of -85.0
- HitPart particles plays at the `Prefabs/Objects/BeeRocket` position
- `b33BotRocketLaunchInAir` sound plays
- Over the course of 41.0 frames, the `Prefabs/Objects/BeeRocket` moves to (this enemy x position - 1.0, 10.0, 0.0) via a SmoothLerp. After the  first frame yield past 20.0 frames animstate set to 104
- `Prefabs/Objects/BeeRocket` position set to (0.0, 20.0, 0.0)
- `Prefabs/Objects/BeeRocket` z angle set to 75.0
- AddDelayedProjectile 1 call happens
- `data[1]` set to 4 (this is the cooldown for launching another delayed projectile)

## Move 1 - Laser attack
A party wide laser attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- Camera moves to look near the midpoint between this enemey and `partymiddle`
- animstate set to 102
- `B33BotHatchOpen` sound plays
- Yield for 0.65 seconds
- animstate set to 103
- Yield for a frame
- `anim` gets disabled
- `b33BotLaser` sound plays
- Done 2 times:
    - `model`'s second or third child (their front or back canon)'s z angle set to 75.0. The front canon is changed on the first iteration and the back one is changed on the second
    - A new `Prefabs/Particles/Lazer` GameObject is instantiated childed to the matching canon with a local postion of (-1.0, 0.0, 0.0), a y angle of -95.0 and their ParticleSystem's main module's startLifetime set to a 15.0 constant curve
    - The `Prefabs/Particles/Lazer`'s ParticleSystem's MainModule is fast forwarded via Simulate for 20.0 seconds
- Yield for 0.2 seconds
- Over the course of 26.0 frames, both canon's z angle changes from 75.0 to 10.0 via LerpAngle
- Both `Prefabs/Particles/Lazer` gets destroyed in 0.15 seconds
- Done 4 times:
    - `b33BotLaserExplosions` sound plays with a pitch of 1.0 + 0.1 * the index of this iteration starting at 0
    - `ExplosionPillar` particles played at a specific lerp between this enemy -1.0 in x and `partymiddle` where the factor is the (this iteration index starting at 1) / 4 (each gets closer to `partymiddle` with equidistance between the lerp factors)
    - ShakeScreen called with ammount of 0.05 * this iteration index starting at 1 and 0.2 time
    - Yield for 0.2 seconds if this isn't the last iteration
    - `anim` is enabled
    - animstate set to 104
- PartyDamage 1 called
- Yield for 0.5 seconds
- `Click2` sound plays

## Move 2 - Missiles throw
A multiple targets missiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 2 times (3 times instead if hardmode is true), but the call and each subsequent ones won't happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable returns -1|2|null|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|A new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at this enemy's front canon position (back canon instead if it's the second call) + (-1.0, 0.5, -0.2) with scale (0.5, 0.75, 1.0)|50.0 (40.0 instead if hardmode is true)|4.0|null|`deathsmoke`|`Explosion`|null|Vector3.zero|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- Camera moves to look near the midpoint between this enemey and `partymiddle`
- animstate set to 100
- `B33BotHatchOpen` sound plays
- Yield for 0.65 seconds
- The amount of hits is determined which is always 2 (3 instead if hardmode is true)
- animstate set to 101
- Yield for a frame
- `anim` gets disabled
- A new array of GameObject is created with the length being the amount of hits to do to hold the projectiles
- A new sprite object is created rooted using the `guisprites[93]` sprite (a crosshair) positioned offscreen at 99.0 in y with a pure green color on layer 15 (`3DUI`)
- For each hits to do:
    - [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable is called
    - If it didn't returned -1 (a valid target was found):
        - The crosshair scale is set to 3x
        - GradualScale happens on the crosshair towards 1x with 15.0 frametime without destroy
        - `b33BotSummonNoise` sound plays using `sounds[5]` with 1.3 pitch
        - The crosshair position is set to the target player party member's position + (0.0, 1.5, 0.1)
        - Yield for 0.1 seconds
        - `sounds[5]` stopped
        - `b33BotInstantMissle` sound plays
        - The projectile element is set to a new `Prefabs/Objects/BeeRocket` GameObject rooted positioned at this enemy's front canon position (back canon instead if it's the second call) + (-1.0, 0.5, -0.2) with scale (0.5, 0.75, 1.0)
        - HitPart particles plays at the projectile position
        - Projectile 1 call happens
        - Yield for 0.85 seconds (0.65 seconds instead if hardmode is true)
    - Otherwise (no target was found), for each player party members whose `hp` is 0:
        - The crosshair position is set to the player party member's position + (0.0, 1.0, 0.1)
        - The crosshair scale is set to 3x
        - GradualScale happens on the crosshair towards 1x with 15.0 frametime without destroy
        - `b33BotSummonNoise` sound plays using `sounds[5]` with 1.3 pitch
        - Yield for 0.5 seconds
        - `sounds[5]` stopped
- `anim` enabled
- animstate set to 104
- Yield all frames until all projectiles elements are null (all Projectile calls completed)
- Yield for 0.5 seconds
- `Click2` sound plays
- The crosshair is destroyed

## Move 3 - [BeeBot](BeeBot.md) summon
Summon a new [BeeBot](BeeBot.md) enemy. No damages are dealt.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect anything for this action.

### Logic sequence

- animstate set to 107
- `b33BotSummonNoise` sound plays
- Yield for 0.5 seconds
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [BeeBot](BeeBot.md) enemy with type `OffscreenNoAnim` at (0.6, 0.0, -0.75)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- animstate set to 0 (`Idle`)
- `data[0]` set to 3 (this is the cooldown to use this move again)

## Move 4 - Prepares and charge
Prepares an attack next turn which increments `charge`. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect anything for this action.

### Logic sequence

- animstate set to 107
- `b33BotChargept1` sound plays
- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 40.0 frametimer
- Yield for 0.75 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- `charge` incremented
- `basestate` and the local startstate set to 107 (this is what will allow to use move 5 on the next actor turn which is the charged dash attack)
- `data[3]` set to 3 (this is the cooldown to use this move again)
- Yield for 0.5 seconds

## Move 5 - Dash attack
A party wide charged dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|null|`commandsuccess`|20.0|(0.0, 30.0, 0.0)|false|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim)}|

### Logic sequence

- `height` set to 0
- animstate set to 23 (`Chase`)
- y position increased by `height` value before this action
- `b33BotChargept1` sound plays
- Over the course of 61.0 frames, this enemy moves to its current position + (1.0, -0.5, 0.0) via a BeizierCurve3 with a ymax of -1.5
- `b33BotChargept2` sound plays
- ShakeSprite called with intensity (0.15, 0.0, 0.0) and 60.0 frametimer
- Yield for a second
- `model`'s 6th children's ParticleSystem (a thrust below on the right of their main body part) is played
- `model`'s 5th children's ParticleSystem (a thrust below their main body part) is stopped
- The first `shadow`'s ParticleSystem is stopped
- `b33BotChargept3` sound plays
- Over the course of 61.0 frames, this enemy moves to (-30.0, 0.0, 0.0) via a SmoothLerp. On the first frame that this enemy x position becomes less than -4.5:
    - `BigHit` sound plays
    - PartyDamage 1 call happens
    - ShakeScreen called with 0.5 ammount and 0.35 time
- position set to (30.0, 0.0, 0.0)
- `height` set to the value before this action
- animstate set to 0 (`Idle`)
- Over the course of 61.0 frames, this enemy moves to startp via a SmoothLerp
- `model`'s 6th children's ParticleSystem (a thrust below on the right of their main body part) is stopped
- `model`'s 5th children's ParticleSystem (a thrust below their main body part) is played
- The first `shadow`'s ParticleSystem is played
- `basestate`, animstate and the local startstate set to 0 (`Idle`) which will prevent this move to be used again until move 4 is used (charge and prepare)
