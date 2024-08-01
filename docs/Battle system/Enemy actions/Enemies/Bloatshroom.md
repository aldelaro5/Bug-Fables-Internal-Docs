# `Bloatshroom`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the numbing mist move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the numbing mist move when summoning a [Mushroom](Mushroom.md), the summoned enemy will be able to act on the current main turn right after this enemy actor turn is over
- The move selection odds changes to slightly favor the numbing mist move
- In the jump attack move, the amount of frames this enemy takes to reach its target changes to 38.25 frames from 51.0 frames

## Move selection
2 moves are possible:

1. A party wide numbing mist that may either summon a [Mushroom](Mushroom.md) or targets other enemy party members except ones from the fungi family who are healed instead 
2. A single target jump attack

The decision of which move to use is based on odds, but the odds changes based on hardmode being true or not. The following table shows the odds in either situations:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|----|----|
|1|30%|40%|
|2|70%|60%|

## Move 1 - Numbing mist
A party wide numbing mist that may either summon a [Mushroom](Mushroom.md) or targets other enemy party members except ones from the fungi family who are healed instead.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md) calls

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|1|[Numb](../../Damage%20pipeline/AttackProperty.md)|true only if `barfill` is at least 90.0 after DoCommand 1 (meaning the bar was filled at least 90% of the way), false otherwise|0.0|Vector3.zero|false|null|

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Only happens if this enemy isn't the only `enemydata` left at the start of the action and it happens for each enemy party member other than this enemy whose kind isn't from the following list:<ul><li>[CordycepsAnt](CordycepsAnt.md)</li><li>[Mushroom](Mushroom.md)</li><li>[Zombee](Zombee.md)</li><li>[Zombeetle](Zombeetle.md)</li><li>[Bloatshroom](Bloatshroom.md)</li><li>[Zommoth](Zommoth.md)</li></ul>|null|The enemy party member|1 (this cannot be lethal to the target as enforced after the call)|[Numb](../../Damage%20pipeline/AttackProperty.md)|null|false|

### Logic sequence

- If this enemy is the last `enemydata` left:
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (3.5, 0.0, 0.0) with a multiplier of 2.0
    - Yield all frames until `forcemove` is done
    - startp is set to this enemy position with the y component at 0.0
- `Jump` sound plays with 0.8 pitch
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy with an h of `jumpheight` * 1.25
- Yield for 0.5 seconds
- Yield all frames until this enemy is `onground`
- animstate set to 5 (`Angry`)
- `rotater` gets a SpriteBounce added with a `startscale` of true and a MessageBounce with a 2.0 multiplier
- `Mist` sound plays
- `Spores` particles plays at this enemy position + 1.0 in y with -1.0 time (infinite)
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the [TappingKey](../../Action%20commands/TappingKey.md) command's help)
- DoCommand 1 call happens
- All player party member who isn't [stopped](../../Actors%20states/IsStopped.md) has their y `spin` set to 15.0 and their animstate to 11 (`Hurt`)
- Yield all frames until `doingaction` is false (DoCommand 1 is done), Before each frame yield, `flip` is set to false
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- The `Spores` particles is moved offscreen at -9999.0 in y then destroyed in 10.0 seconds
- All player party members has their `spin` zeroed out
- PartyDamage 1 call happens
- The SpriteBounce of the `rotater` gets destroyed
- `rotater`'s scale gets set to the value it had before this action
- If this enemy is the last `enemydata`:
    - `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [Mushroom](Mushroom.md) with type `FromGroundKeepScale` at (3.5, either 3.0 or -3.0 determined with uniform RNG, 0.0) and with cantmove (without cantmove instead if hardmode is true meaning the enemy will only be able to act on the current main turn if hardmode is true)
    - Yield all frames until `checkingdead` is null (the coroutine completed)
- Otherwise (there were already other enemy party members than this enemy), 
    - For each enemy party members other than this enemy:
        - If they are a [CordycepsAnt](CordycepsAnt.md), [Mushroom](Mushroom.md), [Zombee](Zombee.md), [Zombeetle](Zombeetle.md), [Bloatshroom](Bloatshroom.md) or [Zommoth](Zommoth.md), [Heal](../../Actors%20states/Heal.md) is called on them to heal an `hp` amount of their `maxhp` * 0.2 ceiled then clamped from 0 to 5
        - Otherwise (the enemy party member isn't a fungi), DoDamage 1 call happens on them
        - If the enemy party member's `hp` is 1 or below, it is set to 1 (this prevents DoDamage 1 from being lethal to them)
- Yield for 0.5 seconds

## Move 2 - Jump attack
A single target jump attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- `checkingdead` is set to a new HeavyJump coroutine for this enemy with 3 damage and 60.0 speed (45.0 if hardmode is true)
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine effectively ends up doing:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.1)
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped (this won't do anything since no sounds should be playing there)
- Yield for 0.3 seconds
- `Jump` sound plays with 0.8 pitch
- `Turn` sound plays with 0.7 pitch
- Over the course of 51.0 frames (38.25 frames instead if hardmode is true), this enemy moves to `playertargetentity` position -0.1 in z via a BeizierCurve3 with a ymax of 5.5 scaled over the course of 60.0 frames (45.0 frames instead if hardmode is true). Before each frame yield, animstate is set to 2 (`Jump`) until halfway 30.0 frames passed (27.5 frames instead if harmode is true) where it's set to 3 (`Fall`) instead
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE), ShakeScreen is called with ammount (0.075, 0.3, 0.0) and 0.75 time with dontreset
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE):
    - Over the course of 9.0 frames (16.75 frames instead if hardmode is true), this enemy moves to `playertargetentity` position -0.1 in z via a BeizierCurve3 with a ymax of 5.5. Before each frame yield, animstate is set to 3 (`Fall`)
    - `impactsmoke` particles plays at `playertargetentity` position
    - `Thud4` sound plays
    - `HugeHit3` soud plays
- Otherwise (blocked, ignores FRAMEONE):
    - Over the course of 41.0 frames, this enemy moves to `playertargetentity` position + (2.0, 0.0, -0.1) via a BeizierCurve3 with a ymax of 3.0. Before each frame yield, animstate is set to 3 (`Fall`)
    - `Thud4` sound plays
    - `impactsmoke` particles plays at `playertargetentity` position
    - ShakeScreen is called with ammount (0.05, 0.2, 0.0) and 0.5 time with dontreset
    - animstate set to 0 (`Idle`)
- Yield for 0.5 seconds
- `checkingdead` is set to null informing the caller that the coroutine completed
