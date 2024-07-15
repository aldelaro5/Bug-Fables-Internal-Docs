# `SeedlingKing`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: When the value is 0, it means this enemy hasn't went through its phase transition yet which happens on the start of the actor turn if their [HPPercent](../../Actors%20states/HPPercent.md) reaches 0.5 or below. When this transition occurs, the value is set to 1 so the transition only happens once per battle

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 3 changes:

- When the phase transition occurs in the pre move logic, this enemy gains a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition for 9999999 main turns (effectively infinite)
- In the enemy summon move, the summoned enemy gains an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) and a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions for 999999 each (effectively infinite)
- If this enemy attempts to use the enemy summon move when there's already 3 or more enemy party members and the phase transition hasn't happened yet a seeds throw move will be used instead of not using any moves

## Move selection
4 moves are possible:

1. A single target tackle attack
2. A party wide rolling attack
3. Summons up to 2 new enemies from the seedlings family of enemies
4. Multiple seeds throws targetting random player party members

The move usage is based on odds, but those odds changes if [HPPercent](../../Actors%20states/HPPercent.md) becomes 0.5 or lower. Here are the base odds of each move in either situations:

|Move|Odds when [HPPercent](../../Actors%20states/HPPercent.md) > 0.5|Odds when [HPPercent](../../Actors%20states/HPPercent.md) <= 0.5|
|---:|------------------------------|------------------------------|
|1|Never used|4/11|
|2|Never used|3/11|
|3|3/4|2/11|
|4|1/4|2/11|

However, move 3 will not be used if there are already 3 or more enemy party members. What happens in that situation when it is selected is decided by the following logic:

- If [HPPercent](../../Actors%20states/HPPercent.md) is 0.5 or lower, Move 1 is used
- Otherwise, if hardmode is false, the entire action ends with no moves being used
- Otherwise (hardmode is true), move 4 is used

## Pre move logic
The following logic always happen at the start of the actor turn.

The game determines if we are before or after phase transition threshold by checking if [HPPercent](../../Actors%20states/HPPercent.md) is 0.5 or below. If it is and `data[0]` is 0, it means the transition has yet to happen. Notably, it sets `moves` to 2 which grants 2 actor turns per main turns to this enemy (excluding the current main turns). This is the phase transition logic:

- If hardmode is true, [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the new enemy for 9999999 main turns (effectively infinite)
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md) called with emote `Exclamation` for 50 time
- `Wam` sound plays
- `data[0]` set to 1 so this transition doesn't happen again
- animstate set to 100
- `Prefabs/Particles/AngryParticle` instantiated childed to the `sprite` with a local position of `freezeoffset`
- Yield for 0.75 seconds
- `Prefabs/Particles/AngryParticle` gets destroyed
- animstate set to 2 (`Jump`)
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md) called with an h of 15.0
- `Turn` sound plays with 0.9 pitch
- Yield for 0.5 seconds
- `height` and `minheight` set to 0.0
- `subentity` set to null after saving their elements into a local array
- All previous `subentity` elements gets rooted and their animstate set to 11 (`Hurt`)
- Yield until this enemy is `onground`
- animstate set to 0 (`Idle`)
- SetAnim called with `f`
- `Thump5` sound plays
- `impactsmoke` particles plays at this enemy position
- For each older `subentity` elements:
    - x angles set to 85.0
    - FadeSprite called with 90.0 frametime with destroy
- ShakeScreen called with 0.2 ammount and 0.75 time
- Yield for 0.75 seconds
- `moves` set to 2 (meaning this enemy now gets 2 actor turns per main turns)
- SetAnim called with empty string
 
## Move 1 - Tackle attack
A single target tackle attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|null|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- animstate set to 1 (`Walk`)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.1) with 2.0 multiplier using both 1 (`Walk`) as walkstate and stopstate
- Yield all frames until `forcemove` is done
- `Grow1` sound plays with 0.7 pitch
- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 45.0 frametimer
- This enemy moves +1.0 in x via MainManager.MoveTowards with 45.0 frametime with smooth with local
- Yield for 0.75 seconds
- `Grow2` sound plays
- Over the course of 10.5 frames, this enemy moves to `playertargetentity` position + (0.75, 0.0, -0.1) via a lerp
- `HugeHit` sound plays
- ShakeScreen called with ammount (0.2, 0.0, 0.0) with 0.75 time with dontreset
- DoDamage 1 call happens
- Over the course of 31.0 frames, this enemy moves +1.5 in x via a BeizierCurve3 with a ymax of 3.0
- `ThumpSoft` sound plays

## Move 2 - Rolling attack
A party wide rolling attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|null|`commandsuccess`|10.0|(0.0, 20.0, 0.0)|false|null|

### Logic sequence

- ShakeSprite called with intensity (0.1, 0.0, 0.0) and 60.0 frametimer
- This enemy moves +1.0 in x via MainManager.MoveTowards with 60.0 frametime with smooth without local
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- `Spin5` sound plays on loop using `sounds[5]`
- Yield for a second
- `trail` set to true
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `sounds[5]` stopped
- `SeedlingKingRoll` sound plays
- Over the course of 37.5 frames, this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame their x position gets less than -4.5:
    - ShakeScreen called with 0.2 ammount and 0.75 time
    - DoDamage 1 call happens
- `trail` set to false
- animstate set to 1 (`Walk`)
- Yield for 0.75 seconds
- Over the course of 38.0 frames, this enemy moves from (10.0, 0.0, 0.0) to startp via a lerp

## Move 3 - Enemies summon
Summons up to 2 new enemies from the seedlings family of enemies.

### Logic sequence

- animstate set to 100
- `rustling1` sound plays
- Yield for 0.25 seconds
- The position of the new enemies is determined depending on if their place is taken or not. The left location is at (0.25, 0.0, -0.1) and the right location is at (7.0, 0.0, 0.1). The left one takes priority if available, but both will be summoned if both locations are free
- Yield for 0.33 seconds
- For each free location available to summon from the 2 ones above:
    - `rustling1` sound plays
    - A new sprite object is created rooted using the `TangyBerry` [item](../../../Enums%20and%20IDs/Items.md) sprite
    - Over the course of 61.0 frames, the sprite moves to the target location via a BeizierCurve3 with a ymax of 6.0. Before each frame yield, the z angle is set to the frame index * 20.0
    - The sprite is destroyed
    - A seedling family enemy id is generated using the following odds:
        - [Seedling](Seedling.md): 2/7
        - [Acornling](Acornling.md): 2/7
        - [Underling](Underling.md): 2/7
        - [Cactus](Cactus.md): 1/7 (overriden to `Seedling` if this is the left location)
    - `Charge` sound plays
    - `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon the enemy with the generated id with type `FromGroundInstant` at the location without cantmove (meaning this enemy will be able to act immeditely on the current main turn)
    - Yield until `checkingdead` is null (the coroutine completed)
    - The new enemy gets some adjustements:
        - `maxhp`: Divided by 2 floored
        - `hp`: Set to their `maxhp`
        - `exp`: Divided by 2 floored
    - If hardmode is true:
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the new enemy for 999999 main turns (effectively infinite)
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the new enemy for 999999 main turns (effectively infinite)

## Move 4 - Seeds throw
Multiple seeds throws targetting random player party members.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen for half the amount of seeds thrown ceiled, but each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1. The amount of seeds thrown is random between 3 and 5 inclusive|null|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|A new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.1) using the `spritemat` material on layer 0 (Default)|55.0|A random integer between 6 and 8 inclusive which is then cast to float|null|null|`WoodHit`|Empty string|(0.0, 0.0, random integer between -20 and 20 which is then cast to float)|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- ShakeScreen called for 0.05 ammount and -1.0 time
- `animstate` set to 100
- `rustling1` sound plays on loop using `sounds[5]`
- Yield for 0.5 seconds
- `checkingdead` set to a new SeedShake coroutine called in such a way that it effectively ends up doing the following:
    - A SpriteRenderer array is created to contain the seeds with the length being the amount to throw which is random between 3 and 5 inclusive
    - For each seeds to throw:
        - The seed element of the array is initialised to a new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.1) using the `spritemat` material on layer 0 (Default)
        - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable. NOTE: This targetting scheme is broken, see the [nullable GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for details
        - If the seed index is even and the target player party member isn't -1, `Ping` sound plays followed by a Projectile 1 call. If it's odd, `PingUp` sound plays with 0.75 volume followed by the seed object moving via MainManager.ArcMovement to a random vector between (0.0, 0.0, -5.0) and (10.0, 0.0, 5.0) with destroyonend and using the same spin, height and frametime than the Projectile 1 call
        - Yield for a random interval between 0.3 and 0.6 seconds
    - Yield all frames until all elements of the seeds array are null (all Projectile 1 calls completed)
    - `checkingdead` set to null
- Yield all frames until `checkingdead` is null (SeedShake completed)
- MainManager.`screenshake` set to Vector3.zero
- `sounds[5]` stopped
