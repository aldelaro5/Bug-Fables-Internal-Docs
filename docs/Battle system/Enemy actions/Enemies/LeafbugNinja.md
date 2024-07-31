# `LeafbugNinja`

## Assumptions
It is assumed this enemy is loaded with a non empty [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) as it is needed for their `hitaction` logic to work. Typically, it will be set to other `LeafbugNinja`, [LeafbugArcher](LeafbugArcher.md) and [LeafbugClubber](LeafbugClubber.md).

## About the clones special logic
This enemy is able to summon clones of themselves in the post move logic, but with a special property: their [weakness](../../Actors%20states/Enemy%20features.md#weakness) is set to be one element being [DieInOneHit](../../Damage%20pipeline/AttackProperty.md). This not only changes the damage pipeline such that their `hp` is unconditionally set to 0 upon any [DoDamage](../../Damage%20pipeline/DoDamage.md) calls, but it allows this action to tell that they are indeed a clone. This assumes that this enemy's data does NOT contain this property in the `weakness` field as it would otherwise break this system.

Clones have very reduced logic as besides the [hitaction](../../Actors%20states/Enemy%20features.md#hitaction) logic and the initialisation of `data` (which won't do anything for them), they do not share the logic of the original enemy who summoned them. If at the start of the action, their `hitaction` is false, but they have the `DieInOneHit` in their `weakness`, the ENTIRE action logic is reduced to this:

- The nocharm local is set to true which will prevent any [UseCharm](../../Battle%20flow/UseCharm.md) calls to occur in the post action phase. This means no `HealHP` charm can occur after this action

This means the clones cannot perform any moves, summons other clones or do anything practical besides their `hitaction` logic. Only the original enemy who summoned them can have access to the full action logic.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: When this is 0, the cloning setup won't be done in the post move logic unless this enemy is the last remaining enemy party member. Once the cloning setup is performed for the first time, it is set to 1. When this happens, the cloning setup will always be performed in the post move logic even if there are more than 1 enemy party members including this enemy

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the leaf projectiles throw move, there is now a 5/10 chance for the Projectile property to be [Poison](../../Damage%20pipeline/AttackProperty.md) instead of [Pierce](../../Damage%20pipeline/AttackProperty.md). When this happens, the projectile has a color of pure magenta
- In the leaf projectiles throw move, the speed of each Projectile changes to 27.0 from 36.0

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves. This logic is supported by the clones, but it's only for visual effects in this case as the `charge` gained won't be useful to them.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- If `charge` is less than 3, it is incremented with the following:
    - animstate set to 5 (`Angry`)
    - `Wam` sound plays
    - [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote with 20 time
    - Yield all frames until `emoticoncooldown` expires
    - `StatUp` sound plays
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
    - Yield for 0.5 seconds
    - `charge` is incremented

## Move selection
2 moves are possible:

1. A single target kick attack
2. A multiple targets leaf projectiles throw

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

## Post move logic
The following logic concerns the cloning setup and it is always done after a move, but only if at least one of the following is true:

- This enemy is the last `enemydata` remaining
- `data[0]` is 1 (meaning this cloning setup was used before)

If these requirements are fufilled, the cloning logic happens:

- `charge` set to 0
- [UpdateEntities](../../Visual%20rendering/UpdateEntities.md) called
- `data[0]` set to 1 (allows this cloning setup to be done again even if there are more than 1 enemy party members)
- If there are less than 3 `enemydata`:
    - animstate set to 104
    - Yield for 0.5 seconds
- Done 2 times as long as there are less than 3 `enemydata`:
    - `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon another `LeafbugNinja` with type `None` at a random location between (0.25, 0.0, -0.75) and (7.0, 0.0, 0.75) where the distance between any enemy party member is less than its `size` * 1.75 with cantmove (meaning the enemy needs to wait on the next main turn to act)
    - Yield all frames until `checkingdead` is null
    - `LeafClone` sound plays
    - DeathSmoke particles plays at `enemydata[lastaddedid]` position with a size of (1.5, 2.5, 1.5)
    - The fields of `enemydata[lastaddedid]` gets adjusted:
        - [weakness](../../Actors%20states/Enemy%20features.md#weakness): [DieInOneHit](../../Damage%20pipeline/AttackProperty.md)
        - `hp`: This enemy's `hp`
        - `destroytype`: [NinjaLog](../../../Entities/EntityControl/Notable%20methods/Death.md#ninjalog)
        - `money`: 0
        - `exp`: 0
        - [diebyitself](../../Actors%20states/Enemy%20features.md#diebyitself): true
        - `charge`: This enemy's `charge`
        - [condition](../../Actors%20states/Conditions.md): This enemy's `condition`
    - Yield for 0.5 seconds
- [UpdateEntities](../../Visual%20rendering/UpdateEntities.md) called
- `LeafCloneShuffle` sound plays
- From there, there are 3 position vectors that the 3 enemy party members will end up, but who will go where is determined randomly with this enemy being the first whose position will be determined. Here are the 3 position vectors:
    - (1.0, 0.0, 0.0)
    - (3.5, 0.0, 0.0)
    - (6.0, 0.0, 0.0)
- startp is set to a random position vector among the 3 which determines the position vector of this enemy. This also determines the position vector of the other 2 enemy party members: the first free position vector will be assigned to the first `enemydata` that isn't this enemy and the last remaining one will be assigned to the remaining `enemydata`
- For every enemy party members:
    - If this isn't this enemy, their `hp` is set to this enemy's `hp`
    - They [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (3.5, 0.0, 0.0) with a 2.0 multiplier and 1 (`Walk`) as walkstate and stopstate
    - Their y `spin` set to 30.0
- Yield all frames until the first 3 `enemydata`'s `forcemove` are all done
- Yield for 0.5 seconds
- For each enemy party members:
    - Their `spin` gets zeroed out
    - Their `flip` gets set to false
    - They get FlipAngle called with setangle
    - They [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) their respective position vector with a 2.0 multiplier
- Yield all frames until the first 3 `enemydata`'s `forcemove` are all done
- This enemy position set to startp
- For every enemy party members, their `charge` is reset to 0

## Move 1 - Kick attack
A single target kick attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.25, 0.0, -0.1) with a 3.0 multiplier
- Yield all frames until `forcemove` is done
- `ChargeDown2` sound plays with 1.1 pitch
- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 101
- `ChargeDown2` sound plays
- `Hit3` sound plays
- DoDamage 1 call happens
- `LeafFlipBack` sound plays
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 102
- z `spin` set to -30.0
- Over the course of 41.0 frames, this enemy moves from its position + 1.0 in y to startp + 1.0 in y via a BeizierCurve3 with a ymax of 5.5
- `spin` zeroed out
- animstate set to 1 (`Walk`)
- FlipAngle called with setangle
- `sprite` angle zeroed out
- position set to startp
- Yield for 0.5 seconds

## Move 2 - Leaf projectiles throw
A multiple targets leaf projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but each calls requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup> (if hardmode is true, this has 5/10 chance to be [Poison](../../Damage%20pipeline/AttackProperty.md) instead)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|A new sprite object childed to the `battlemap` using the `projectilepsrites[13]` sprite (leaf projectile) positioned at this enemy + (-1.0, 1.0, -0.1) with a scale of 0.65x and a z angle of -90.0. If the property is [Poison](../../Damage%20pipeline/AttackProperty.md), the SpriteRenderer's material color is pure magenta|36.0 (27.0 instead if hardmode is true)|0.0|null|null|null|null|Vector3.zero|false|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- animstate set to 100
- Yield for 0.5 seconds
- Done 3 times as long as at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - animstate set to 103
    - `Toss3` sound plays
    - A new sprite object is created childed to the `battlemap` using the `projectilepsrites[13]` sprite (leaf projectile) positioned at this enemy + (-1.0, 1.0, -0.1) with a scale of 0.65x and a z angle of -90.0
    - If hardmode is true and a 5/10 RNG check passes, the property of the Projectile 1 call will be `Poison` and the sprite object's SpriteRenderer's material color is set to pure magenta
    - Projectile 1 call happens
    - Yield for 0.25 seconds
    - animstate set to 100
    - Yield all frames until the sprite object is null (Projectile 1 call completed)
    - animstate set to 100
    - Yield for 0.25 seconds
- Yield for 0.5 seconds
