# `Abomihoney`

## `data` usage
During the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0. An unitialised value is possible, but it will be treated the same was as if it was 0.

- `data[0]`: This remains at 0 until the bomb transformation move is used where it's set to 1. The value being 1 means the bomb explosion move will be used on the next actor turn

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The RNG check required to use the [Ahoneynation](Ahoneynation.md) healing move when all other conditions are fufilled has its odds changed to 51% from 41%
- The `hp` / `maxhp` floored threshold required to use the bomb transform move when all other conditions are fufilled changes to being below 0.75 from 0.5 (meaning 75% `hp` remaining instead of 50%)
- The RNG check required to use the bomb transform move when all other conditions are fufilled has its odds changed to 41% from 31%
- In the bomb transformation move, this enemy gets a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 99999 main turns (effectively infinite)

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves

### Logic sequence

- If [position](../../Actors%20states/BattlePosition.md) is `Ground` and either `data[0]` is 0 or wasn't initialised yet (meaning the bomb explosion move isn't slated to be used on the next actor turn):
    - animstate set to 104
    - `Puddle` sound plays
    - Yield for 0.5 seconds
    - `digging` set to true
    - animstate set to 0 (`Idle`)
    - [position](../../Actors%20states/BattlePosition.md) set to `Underground`
    - The local startstate is set to 31 (`Dig`)

## Move selection
1 moves are possible:

1. A single target grounded bite attack
2. A single target underground bite attack
3. Sacrifice themselves to heal all their remaining `hp` to a [Ahoneynation](Ahoneynation.md)
4. Turns into a bomb that will explode on their next actor turn
5. Explode as a bomb which kills themselve, but  dealing damages to everyone except other other `Abomihoney` and [Ahoneynation](Ahoneynation.md) which gets healed

Every moves requires specific conditions to be used and some of them requires an RNG check. Here is the decision process done in order:

- If [position](../../Actors%20states/BattlePosition.md) is `Underground`, move 2 is used
- Otherwise, if all of the following conditions are fufilled, move 3 is used:
    - A [Ahoneynation](Ahoneynation.md) is present
    - The first [Ahoneynation](Ahoneynation.md) in `enemydata` has an `hp` / `maxhp` floored below 0.6 (less than 60% `hp` remaining)
    - `locktri` is true (this can only happen under normal gameplay if this enemy was summoned by [Ahoneynation](Ahoneynation.md))
    - A 41% RNG check passes (51% instead if hardmode is true)
- Otherwise, if `data[0]` is 1 (meaning move 4 was used on their last actor turn), move 5 is used
- Otherwise, move 4 is used if all of the following conditions are fufilled:
    - `locktri` is false (meaning this isn't an enemy summoned by [Ahoneynation](Ahoneynation.md))
    - `hp` / `maxhp` floored is less than 0.5 (0.75 instead if hardmode is true). This means less than 50% (or 75%) `hp` remaining
    - A 31% RNG check passes (41% instead if hardmode is true)
- Otherwise (no other moves was slated for usage), move 1 is used

## Move 1 - Grounded bite
A single target grounded bite attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (1.5, 0.0, -0.1) with 1.0 multiplier (2.0 instead if this enemy x position is higher than 3.0)
- `PuddleMove` sound plays via PlayMoveSound on loop using `sounds[9]` with 1.15 pitch
- Yield all frames until `forcemove` is done
- `sound` is set to stop looping and stopped
- Yield for a frame
- animstate set to 100
- `Bite3` sound plays
- Yield for 0.65 seconds
- animstate set to 102
- `Bite2` sound plays
- DoDamage 1 call happens
- Yield for 0.7 seconds

## Move 2 - Underground bite
A single target underground bite attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- `PuddleMove` sound plays on this enemy
- `sound` is set to loop
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (0.0, 0.0, -0.1) with 1.6 multiplier
- Yield all frames until `forcemove` is done
- `sound` is set to stop looping and stopped
- animstate set to 106
- `Bite4` sound plays
- Yield for 0.1 seconds
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called with an h of 15.0
- DoDamage 1 call happens
- Yield for 0.75 seconds
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) + 2.75 in x with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 105
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)
- [position](../../Actors%20states/BattlePosition.md) set to `Ground`
- The local startstate is set to 0 (`Idle`)

## Move 3 - Sacrifice and heals a [Ahoneynation](Ahoneynation.md)
Sacrifice themselves to heal all their remaining `hp` to a [Ahoneynation](Ahoneynation.md). No damages are dealt.

### Logic sequence

- The `enemydata` index of the first [Ahoneynation](Ahoneynation.md) is obtained
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md) called on this enemy
- `Jump` sound plays
- y `spin` set to 10.0
- Yield for 0.5 seconds
- animstate set to 108
- `Puddle` sound plays
- Yield all frames until this enemy is `onground`
- Over the course of 31.0 frames, this enemy moves to the `Ahoneynation` position + 0.1 in z via a BeizierCurve3 with a ymax of 3.0
- [Heal](../../Actors%20states/Heal.md) called to heal the `Ahoneynation`'s `hp` by this enemy's `hp`
- `selfsacrifice` set to true which prevents some logic to happen in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) since this enemy is set to die soon
- [CleanKill](../../Actors%20states/CleanKill.md) called on this enemy which turns this enemy into a blank actor shell placing their battleentity at startp

## Move 4 - Bomb transformation
Turns into a bomb that will explode on their next actor turn. No damages are dealt.

### Logic sequence

- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called with an h of 15.0
- `Jump` sound plays
- y `spin` set to 15.0
- Yield for 0.3 seconds
- `poisonres`, `numbres`, `sleepres` and `freezeres` are all set to 110 (all immune)
- `Fuse` sound plays with 1.1 pitch
- If hardmode is true, [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called on this enemy to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 99999 main turns with effect
- `originalid` and `animid` of this enemy's entity set to 217 (`Abomihoney`). This shouldn't change anything under normal gameplay
- animstate set to 0 (`Idle`)
- `data` is initialised to be one element with a value of 1 which means the bomb explosion move will be used on the next actor turn
- A SpriteBounce is added to `rotater` with a MessageBounce with a 1.5 multiplier and a `basescale` of 1.25x
- Yield all frames until this enemy is `onground`. Before each frame yield, the `basescale` of the SpriteBounce is set to 1.25x
- `destroytype` set to [ExplodeAnim](../../../Entities/EntityControl/Notable%20methods/Death.md#explodeanim)
- `onhitaction` set to -1 which should prevent this enemy from performing their `hitaction` logic
- `spin` zeroed out
- angles zeroed out
- Yield for 0.5 seconds
- The `basescale` of the SpriteBounce is set to 1.25x

## Move 5 - Bomb explosion
Explode as a bomb which kills themselve, but deals damages to everyone except other other `Abomihoney` and [Ahoneynation](Ahoneynation.md) which gets healed.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen to each player party members whose `hp` is above 0|null|The player party member|10 (5 instead if the player party member has the [Shield](../../Actors%20states/BattleCondition/Shield.md) which also will make the attack non lethal after this call)|null|null|false|
|2|Always happen to every enemy party members except this enemy, other `Abomihoney` and other [Ahoneynation](Ahoneynation.md)|null|The enemy party member|10|null|null|false|

### Logic sequence

- The `Hold` animation clip is played on `anim`
- `Fuse` sound plays
- Over the course of 61.0 frames, the `basescale` of the `rotater`'s SpriteBounce changes to 2x via a lerp
- ShakeSprite called with an intensity of (0.1, 0.0, 0.0) and 40.0 frametimer
- Yield for 0.66 seconds
- `explosion` particles plays at this enemy position
- `Explosion3` sound plays
- `Splat2` sound plays
- `sprite` gets disabled
- `shadow` gets disabled
- ShakeScreen called with an ammount of 0.25 and time of 0.75
- For each player party members:
    - If their `hp` is above 0, DoDamage 1 call happens
    - If they had the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition before the DoDamage 1 call, their `hp` gets clamped from 1 to their `maxhp` which makes the DoDamage 1 call non lethal
- For each enemy party member other than this enemy:
    - If it's an [Ahoneynation](Ahoneynation.md) or another `Abomihoney`, [Heal](../../Actors%20states/Heal.md) is called on them to heal 10 of their `hp`
    - Otherwise, DoDamage 2 call happens on them
- 3 new sprite objects are created and appears with the following properties:
    - Named `splatX` where `X` is a number from 0 to 2
    - Childed to the `GUICamera`
    - sprite of `Sprites/Objects/splatter`
    - Positioned in one of 3 hardcoded Vector3
    - Scale of Vector3.zero
    - size of one of 3 hardcoded Vector3
    - color of FFFFBF (mostly orange)
    - sortorder of -5 - X where X is a number from 0 to 2
    - flipX of false except for the second sprite where it's true
    - A DialogueAnim is added with a `targetscale` of Vector3.one and a `shrinkspeed` of 0.15
    - Yield for 0.025 seconds after each creation
- Over the course of 100.0 frames, each `Sprites/Objects/splatter` color's alpha changes to 0.0 via a lerp, but only over on the last 50.0 frames of the full yields (the first 50.0 are just frame yields)
- `sprite` gets disabled
- All `Sprites/Objects/splatter` gets destroyed
- Yield for 0.5 seconds
- `selfsacrifice` set to true which prevents some logic to happen in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) since this enemy is set to die soon
- [CleanKill](../../Actors%20states/CleanKill.md) called on this enemy which turns this enemy into a blank actor shell placing their battleentity at startp
