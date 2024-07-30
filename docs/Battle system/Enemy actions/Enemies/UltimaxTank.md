# `UltimaxTank`

## Assumptions
It is assumed this enemy's [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) is set to its [matching EventDialogue](../../Battle%20flow/EventDialogues/UltimaxTank.md).

It is also assumed this enemy is loaded with the `UltimaxTank` [animid](../../../Enums%20and%20IDs/AnimIDs.md) as the presence of `extra` elements are assumed which are normally initialised by [AnimSpecificQuirk](../../../Entities/EntityControl/Animations/AnimSpecific.md#animspecificquirks) for this animid.

It is also assumed this enemy is initially fought alone so their summoning move logic works in expected ways.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the canonballs throw move, the amount of hits to do changes to 3 from 2
- In the canonballs throw move, the amount of frames each canonball takes to reach its target changes to 21.0 frames from 29.0 frames
- In the canonballs throw move, the damageammount of each DoDamage calls changes to 3 from 4
- In the canonballs throw move, each canonballs now inflicts the [AttackDown](../../Player%20actions/Skills/AttackDown.md) condition to the target if `commandsuccess` is false after the DoDamage call (didn't blocked, ignores FRAMEONE) and the target doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition
- In the bomb throw move, the amount of frames the bomb moves to reach its target changes to 37.0 frames from 44.5 frames
- In the spinning dash attack move, the amount of frames this enemy moves changes to 71.0 frames from 91.0 frames
- In the thrusting dash attack move, the amount of frames this enemy moves changes to 36.0 frames from 46.0 frames

## Move selection
6 moves are possible:

1. A multiple targets canonballs throws
2. A party wide bomb throw of a random type
3. A party wide spining dash attack
4. Prepares themselves for a big attack with 2 `charge` and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
5. Summons a new [WaspTrooper](WaspTrooper.md) or [WaspHealer](WaspHealer.md)
6. A party wide thrusting dash attack

Move 6 is always (and only) used if `basestate` is 104 (meaning move 4 was used before since it sets it to that value).

Move 1 and 2 are treated as the same during the move selection process. If it's selected, what determines the move used between the 2 is `locktri` (this is only true after the phase transition in the pre move logic happened). If it's false, move 1 is used and if it's true, move 2 is used. They will be refered to as "move 1-2" in the move selection process below.

As for the decision of which move to use among move 1-2 through 5, it is based on odds. However, some moves have additional requirements that must be fufilled for the move to be used if selected. If the move is selected, but their requirements aren't fufilled, either another move will be used or a continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection. The following table contains all these informations for every moves:

|Move|Odds|Requirements|Falback if requirements aren't fufilled|
|---:|---|-----|-----|
|1-2|3/10|None, the move is always used|N/A|
|3|2/10|[HPPercent](../../Actors%20states/HPPercent.md) is 0.8 or below|Move 1-2 is used|
|4|2/10|[HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or below|Move 1-2 is used|
|5|3/10|This enemy is the last remaining `enemydata` and `turns` is above 0 (past the first main turn of the battle)|Continue directive issued to the enemy action loop which restarts the entire action and rerolls the move selection process|

## Pre move logic
The following logic always happen before a move is used if `locktri` is false (this logic hasn't happened before) and [HPPercent](../../Actors%20states/HPPercent.md) is below 0.5. If those requirements are met, a phase transition happens:

- animstate set to 11 (`Hurt`)
- MainManager.GradualScale called on `model`'s 0-0-1-2 child (the canon's pipe) to change their scale to (0.85, 1.3, 1.0) with 60.0 frametime without destroy (done in paralele)
- `UltimaxPartExplodeShake` sound plays
- Done 3 times:
    - DeathSmoke particles plays at this enemy + (1.0 + a random number between -2.0 and 2.0, 2.0, -1.5)
    - Yield for 0.5 seconds
- `UltimaxPartExplode` sound plays
- `explosionsmall` particles plays at this enemy position (-2.25, 3.0, 0.0)
- `model`'s 1-3 child's ParticleSystem (some some particles flowing up) plays
- `model`'s 0-0-1-2 child (the canon's normal pipe) is disabled
- `model`'s 0-0-1-4 child (the canon's broken pipe) is enabled
- `locktri` set to true (this prevents this logic from happening again for this battle)
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)
- Yield for 0.25 seconds

## Post move logic
The following logic happens after a move is used only if all of the following conditions are met:

- `turns` % 2 is 0 (every 2 main turns starting from the first one)
- `basestate` isn't 0 (the charging move wasn't used)
- [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1. NOTE: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details. Also, this is separate from the AddDelayedProjectile 1 calls mentioned requirement below. The actual return isn't used

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Happens up to 2 times, but each calls requires that a [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) call with nullable doesn't return -1<sup>1</sup>|A new `Prefabs/Objects/BeeRocket` GameObject rooted whose SpriteRenderer's sprite uses the `projectilepsrites[17]` sprite (a brown missile projectile) positioned at (0.0, 20.0, 0.0) and with a z angle of 75.0|A random `playerdata` with uniform probability among the elements (NOTE: all [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) calls with nullable do NOT have their returns used to determine the target, only to check that the call may happen)|4|2|0|null|60.0|This enemy|`Explosion3`|`explosion`|`MisseleFall`|

1: This is separate from the post move logic requirement. NOTE: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details. The actual return value isn't used, the target is determined separately from this call, but it actually makes it more broken because it not only disregard the player party formation, but it also disregard `forceattack`.

### Logic sequence

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 multipluer
- Yield all frames until `forcemove` is done
- Camera moves to look near (3.0. 1.0, 0.0)
- Yield for 0.25 seconds
- For each hits (2 times):
    - [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable call and if it returns -1, the hit is skipped
    - animstate set to 101
    - A new `Prefabs/Objects/BeeRocket` GameObject rooted whose SpriteRenderer's sprite uses the `projectilepsrites[17]` sprite (a brown missile projectile) positioned at the second to last `extra` (the missile launcher of the `model`) position and with a z angle of -60.0
    - DeathSmoke plays at the `Prefabs/Objects/BeeRocket` position
    - HitPart particles plays at the `Prefabs/Objects/BeeRocket` position
    - `b33BotRocketLaunchInAir` sound plays
    - Over the course of 41.0 frames, the `Prefabs/Objects/BeeRocket` moves to (this enemy x position - 1.0, 10.0, 0.0) via a SmoothLerp. On the second half, before each yield, this enemy animstate is set to 0 (`Idle`)
    - `Prefabs/Objects/BeeRocket` position set to (0.0, 20.0, 0.0)
    - `Prefabs/Objects/BeeRocket` z angle set to 75.0
    - AddDelayedProjectile 1 call happens

## Move 1 - Canonballs throws
A multiple targets canonballs throws.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 2 times (3 times instead if hardmode is true), but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|4 (3 if hardmode is true)|null|null|`commandsuccess`|

### Logic sequence

- Camera moves to look near (1.75, 0.0, 0.0)
- `UltimaxMove` sound plays
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) + 2.5 in x using 1 (`Walk`) as walkstate and 100 as stopstate
- Yield all frames until `forcemove` is done
- Done for each hit which is 2 times (3 times instead if hardmode is true):
    - If there's no alive player party members (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)), the hit is skipped
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - animstate set to 100
    - MainManager.GradualScale called on the `model`'s 0-0-1-2 child (the canon's pipe) to change its scale to (0.7, 1.0, 1.25) with 20.0 frametime without destroy (done in paralel)
    - `UltimaxCannonBallSuckIn` sound plays
    - Yield for 0.5 seconds
    - `model`'s 0-0-1-2 child (the canon's pipe)'s scale set to (1.25, 1.0, 0.7)
    - MainManager.GradualScale called on the `model`'s 0-0-1-2 child (the canon's pipe) to change its scale to 1.0x with 10.0 frametime without destroy (done in paralel)
    - A new `Prefabs/Objects/RollingRock` GameObject is created rooted positioned at `model`'s 0-0-1-2 child (the canon's pipe) position + (-5.4, -0.3, 0.0) with a scale of 0.5x
    - ShakeScreen called with an ammount of 0.1 and 0.3 time with dontreset
    - animstate set to 101
    - `Prefabs/Objects/RollingRock`'s first material's color is set to 404040 (mostly dark gray)
    - This enemy x position increases by 0.85
    - MainManager.MoveTowards called on this enemy to move them to - 0.85 in x with 20.0 frametime without smooth and without local (done in paralel)
    - `UltimaxCannonBallShot` sound plays
    - `explosionsmall` particles plays at the `Prefabs/Objects/RollingRock` position
    - Over the course of 29.0 frames (21.0 frames instead if hardmode is true), `Prefabs/Objects/RollingRock` moves to `playertargetentity` position + 1.0 in y. Before each frame yield, z angle is increased by 20.0 * MainManager.`framestep`
    - ShakeScreen called with an ammount of 0.1 and 0.6 time with dontreset
    - DoDamage 1 call happens
    - If hardmode is true, `commandsuccess` is false (didn't blocked, ignores FRAMEONE) and `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition:
        - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackDown](../../Player%20actions/Skills/AttackDown.md) condition on `playerdata[playertargetID]` for 2 main turns (effectively 1 since the current main turn ends soon enough) with effect
    - MainManager.ArcMovement called on `Prefabs/Objects/RollingRock` to - 10.0 in x with the y component at 0.0 with a spin of -20.0 in z, 5.0 height and 40.0 frametime with destroyonend
    - Yield for 0.5 seconds

## Move 2 - Bomb throw
A party wide bomb throw of a random type.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md) calls

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2 (4 instead if the bomb thrown is a `SpicyBomb`)|Depends on the bomb thrown:<ul><li>`SpicyBomb` or `BurlyBomb`<sup>1</sup>: null</li><li>`NumbBomb`: [Numb](../../Damage%20pipeline/AttackProperty.md)</li><li>`FrostBomb`: [Freeze](../../Damage%20pipeline/AttackProperty.md)</li><li>`SleepBomb`: [Sleep](../../Damage%20pipeline/AttackProperty.md)</li><li>`PoisonBomb`: [Poison](../../Damage%20pipeline/AttackProperty.md)</li></ul>|`commandsuccess`|0.0|Vector3.zero|false|null|

1: This is unexpected because of an issue where no effects are applied to this bomb type. It was expected to inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md), but that doesn't happen even after this call

### Logic sequence

- `TwistDoorEnter` sound plays
- animstate set to 103
- ShakeSprite called with 0.1 intensity and 3.0 frametimer
- Yield for a frame

The bomb type is then determined with random odds:

- `SpicyBomb`: 1/5
- `PoisonBomb`: 1/5
- `SleepBomb`: 1/5
- `FrostBomb`: 1/5
- `NumbBomb`: 1/5

However, if it's a `SleepBomb`, `FrostBomb` or `NumbBomb` and any of the player party members have the [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) or [Sleep](../../Actors%20states/BattleCondition/Sleep.md) conditions, the result is rerolled to be one of the following with equal chances each:

- `SpicyBomb`
- `PoisonBomb`
- `BurlyBomb`

After determining the bomb type, this is what the logic does:

- A new sprite object is created rooted with the matching [item](../../../Enums%20and%20IDs/Items.md) sprite
- Over the course of 31.0 frames, the item sprite moves from the `model`'s 0-0-1-1 child position (the SpriteRenderer near the lid) + (-0.4, 1.92, -0.1) to + (0.95, 1.73, 0.0) via a lerp, but calculated over 4 frames (further frames won't have any changes, but still have yields)
- `Toss4` sound plays
- `Toss3` sound plays
- Over the course of 44.5 frames (37.0 frames instead if hardmode is true), the item sprite position moves to `partymiddle` via a BeizierCurve3 with a ymax of 6.0. Before each yield, the item sprite's z angle increases by 10.0 * MainManager.`framestep`
- The item sprite is destroyed
- `Explosion` sound plays
- Different effects happens depending on the bomb type (except for `BurlyBomb` where no effects happen):
    - `SpicyBomb`:
        - ShakeScreen called with an ammount of 0.2 and 0.75 time with dontreset
        - `explosion` particles plays at `partmymiddle`
    - `PoisonBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `PoisonEffect` particles plays at `partmymiddle` with 2.0x scale
    - `SleepBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - DeathSmoke particles plays at `partmymiddle` with 3.0x size
    - `FrostBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `mothicenormal` particles plays at `partmymiddle` with 3.0x scale
    - `NumbBomb`:
        - ShakeScreen called with an ammount of 0.1 and 0.75 time
        - `ElecFast` particles plays at `partmymiddle` with 2.0x scale
        - `impactsmoke` particles plays at `partmymiddle`
- PartyDamage 1 call happens
- Yield for 0.5 seconds
- Yield for 0.25 seconds

## Move 3 - Spinning dash attack
A party wide spining dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `UltimaxMove` sound plays
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) this enemy + 5.0 in x with a 2.0 multiplier using 1 (`Walk`) as walkstate and 100 as stopstate
- Camera moves to look near (3.75, 0.0, 0.0)
- Yield all frames until `forcemove` is done
- `flip` set to false
- `UltimaxSpinAttackSqueeze` sound plays
- Over the course of 31.0 frames, `startscale` changes to (0.8, 1.25, 1.0) via a lerp
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `UltimaxSpinAttack` sound plays
- `model`'s 1-1 child's ParticleSystem (smoke particles going outwards) plays
- Over the course of 91.0 frames (71.0 frames instead if hardmode is true):
    - This enemy moves to (-20.0, 0.0, 0.0) via a lerp
    - `startscale` changes to the value before this action via a lerp, but the factor is multiplied by 3 (meaning it will reach 1.0 after a third of the way)
    - y `spin` set to 15.0 * (current frame / total frames * 5.0) clamped from 5.0 to 15.0 (meaning from 75.0 to 225.0 reaching the max after a 1/5 of the way)
    - On the first frame this enemy x position becomes lower than -4.5:
        - ShakeScreen called with 0.1 ammount and 0.5 time with dontreset
        - PartyDamage 1 call happens
- `spin` zeroed out
- FlipAngle called with setangle
- position set to (20.0, 0.0, 0.0)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `UltimaxMove` sound plays
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 4.0 multiplier
- Yield all frames until `forcemove` is done

## Move 4 - Prepares big attack with 2 `charge` and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
Prepares themselves for a big attack with 2 `charge` and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- `UltimaxChargeAttackSqueeze` sound plays
- Camera moves to look near this enemy
- `basestate`, animstate and the local startstate are all set to 104 which allows this enemy to use the thrusting dash attack move on the next actor turn
- ShakeSprite called with 0.2 intensity and 45.0 frametimer
- SahkeScreen called with an ammount of 0.05 and 0.75 time with dontreset
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy for 2 main turns (effectively 1 since the current main turn ends soon) with effect
- Yield for 0.25 seconds
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- Yield for 0.5 seconds
- `charge` set to 2

## Move 5 - Summons a [WaspTrooper](WaspTrooper.md) or [WaspHealer](WaspHealer.md)
Summons a new [WaspTrooper](WaspTrooper.md) or [WaspHealer](WaspHealer.md). No damages are dealt.

### Logic sequence

- `TwistDoorEnter` sound plays
- animstate set to 102
- Camera moves to look near this enemy
- Yield for 0.5 seconds
- `Whistle` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon either a [WaspTrooper](WaspTrooper.md) or [WaspHealer](WaspHealer.md) enemy (determined randomly with uniform odds) with type `Offscreen` at (0.95, 0.0, -1.25)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- `enemydata[lastaddedid]` gets some adjustements:
    - `hp` and `maxhp`: value gets multiplied by 0.5 then floored
    - `exp`: divided by 10 floored

## Move 6 - Thrusting dash attack
A party wide thrusting dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|`commandsuccess`|0.0|Vector3.zero|false|null|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- Camera moves to look near (3.75, 0.0, 0.0)
- ShakeSprite called with 0.1 intensity and 57.0 frametimer
- `UltimaxChargeAttackSqueeze` sound plays
- Over the course of 31.0 frames, this enemy moves to + 4.0 in x via a lerp and `startscale` changes to (0.8, 1.25, 1.0) via a lerp
- `model`'s 1-2 child's ParticleSystem (blue thrust particles) plays
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `UltimaxChargeAttack` sound plays
- Over the course of 46.0 frames (36.0 frames instead if hardmode is true):
    - This enemy moves to (-20.0, 0.0, 0.0) via a lerp
    - `startscale` changes to the value before this action via a lerp, but the factor is multiplied by 3 (meaning it will reach 1.0 after a third of the way)
    - On the first frame this enemy x position becomes lower than -4.5:
        - PartyDamage 1 call happens
- `model`'s 1-2 child's ParticleSystem (blue thrust particles) stops playing
- Yield for 0.25 seconds
- position set to startp + 15.0 in y
- `UltimaxChargeAttackFallBack` sound plays
- Over the course of 21.0 frames, this enemy moves to startp via a lerp
- `startscale` set to (1.25, 0.8, 1.25)
- ShakeScreen called with 0.2 ammount and 1.0 time with dontreset
- `impactsmoke` particles plays at this enemy position
- Over the course of 31.0 frames, `startscale` changes to the value it had before this action via a lerp
- `basestate`, animstate and the local startstate all set to 0 (`Idle`) to prevent this enemy from using this move again on the next actor turn
