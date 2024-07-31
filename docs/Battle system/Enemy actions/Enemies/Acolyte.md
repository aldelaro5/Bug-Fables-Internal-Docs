# `Acolyte`

## Asumptions
It is assumed that this enemy has its [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) set to true because it should only change its [position](../../Actors%20states/BattlePosition.md) through specific ways:

- Using the vine spawn move
- Using the aerial strike move
- Have the `AcolyteVine` enemy that they spawn die

That last one leads to another assumption: `AcolyteVine` and this enemy must have an `eventondeath` set to the proper [EventDialogue](../../Battle%20flow/EventDialogues/Acolyte.md) because this EventDialogue will allow the `AcolyteVine` to die first and to drop this enemy's [position](../../Actors%20states/BattlePosition.md) to `Ground`. This is necessary because there's custom logic that needs to happen which goes beyond the regular fall logic of the damage pipeline.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 2 elements with a starting value of 0.

- `data[0]`: The amount of times the vine spawn move needs to be redirected to a vine attack move before being able to spawn an `AcolyteVine` again. When the vine spawn move is used for the first time, this value is set to 2. From there, everytime the vine spawn move is used, if this value is still higher than 0, it is decremented and the vine attack move is used instead. When it reaches 0, the vine spawn move can be used again if selected. There is an exception to this: once `data[1]` gets set to 1 (when the phase transition will occur), this value will be set to 0 on the same actor turn and the vine spawn move will be used. Essentially, once `data[1]` gets set to 1, this enemy will ignore the existing value and proceed with using the vine spawn move anyway
- `data[1]`: A phase transition tracker with three possible states depending on the value:
    - 0: The initial state before [HPPercent](../../Actors%20states/HPPercent.md) becomes 0.4 or lower. From this state, the value is only set to 1 when [HPPercent](../../Actors%20states/HPPercent.md) reaches 0.4 or lower
    - 1: [HPPercent](../../Actors%20states/HPPercent.md) reached 0.4 or lower, but the phase transition hasn't fully occured yet. It occurs on the same actor turn and when it occurs, the value is set to 2 as well as `data[0]` set to 0 on top of using the vine spawn move
    - 2: The phase transition happened and from this point, this value can no longer influence the action logic

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The prayer move, vine spawn move and vine attack moves have their base odds changed
- The vine attack move may hit twice instead of only once, but for 2 base damage on the first hit instead of 3 (the second hit does 2 base damage if it happens)

## Move selection
5 moves are possible:

1. A prayer that inflicts [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) to themselves
2. Spawns a new `AcolyteVine` enemy and gets on top of it which changes their [position](../../Actors%20states/BattlePosition.md) to `Flying`
3. A single target vine attack that may hit multiple times
4. A single target aerial strike that also changes the [position](../../Actors%20states/BattlePosition.md) to `Ground`
5. Transition to the second phase which spawns a new [AngryPlant](AngryPlant.md) enemy then performs move 2

Move 4 can only be used and will always be used if [position](../../Actors%20states/BattlePosition.md) is `Flying`.

For move 5, it can only be used once in the entire battle and it will always be used if all of the following conditions are fufilled:

- [position](../../Actors%20states/BattlePosition.md) is `Ground` (move 2 wasn't used)
- [HPPercent](../../Actors%20states/HPPercent.md) is 0.4 or lower
- `data[1]` is 0 at the start of the actor turn (meaning move 5 was never used before)

Move 5 marks a phase transition whose state is tracked by `data[1]`. It becomes 1 once all of the above is detected and on the same actor turn, it becomes 2 once the transition is completed which prevents to use the move again.

All the other 3 moves have base odds which are shown in the following table and they differ depending if hardmode is true or not. However, the decision of the move to use may not be the final one. It is possible the move's logic decides to perform a different move instead which is reffered to as a redirection. They typically happen when proceeding with the move is illogical or undesired.

Here are all the move's base odds and their possible redirections:

|Move|Base odds (hardmode is false)|Base odds (hardmode is true)|Possible redirections|
|---:|-----------------------------|---------------------------|---------------------|
|1|2/7|1/3|Move 3 if this enemy already has the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition|
|2|2/7|1/3|Move 3 if `data[0]` is above 0 (set to 2 as part of using this move otherwise). An exception to this is when move 5 (phase transition) is used since the usage of move 5 sets `data[0]` to 0 and then uses move 2. `data[0]` decrements when this redirection occurs|
|3|3/7|1/3|None, the move is always used once selected|

## Pre move logic
The following logic is always performed at the start of the action logic:

- `sprite` local position is reset to Vector3.zero

## Post move logic
The following logic is always performed at the end of the action logic:

- `sprite` local position is reset to Vector3.zero

## Move 1 - Prayer
A prayer that inflicts [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) to themselves. No damages are dealt

### `nonphyscal` set to true
This move always sets `nonphyscal` to true, but it doesn't affect anything for this move.

### Logic sequence

- Camera moves to look near this enemy
- animstate set to 103
- Yield for 0.6 seconds
- animstate set to 105
- Yield for 0.35 seconds
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on this enemy for 3 main turns (effectively 2 since the main turn advances soon after)
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 1 (blue up arrow)
- Yield for 0.75 seconds

## Move 2 - `AcolyteVine` spawn
Spawns a new `AcolyteVine` enemy and gets on top of it which changes the [position](../../Actors%20states/BattlePosition.md) to `Flying`. No damages are dealt.

### About status resistance
This move will make this enemy immune to [Poison](../../Actors%20states/BattleCondition/Poison.md), [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md) conditions

### `nonphyscal` set to true
This move always sets `nonphyscal` to true, but it doesn't affect anything for this move.

### Logic sequence

- If `data[0]` is above 0:
    - `data[0]` is decremented
    - Move 3 (vine attack) is performed instead
- Camera moves to look near this enemy
- `data[0]` set to 2
- animstate set to 103
- Yield for 0.6 seconds
- animstate set to 101
- Yield for 0.3 seconds
- `Charge9` sound plays
- ShakeScreen called with an ammount of (0.1, 0.05, 0.0) for 0.5 time
- [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add a new `AcolyteVine` enemy at this enemy position + (1.1, 45.0, 0.1)
- Yield a frame
- `DirtExplode` particles played at the `AcolyteVine` enemy
- Over the course of 30.0 frames, `AcolyteVine` moves to + 5.0 in y with their scale changed to their `startscale` from [endata](../../../TextAsset%20Data/Entity%20data.md#entity-data) via a lerp and their y angle changes from 720.0 to 0.0 via a lerp
- Yield for 0.1 seconds
- animstate set to 100
- Yield for 0.8 seconds
- `Jump` sound plays
- animstate set to 2 (`Jump`)
- y `spin` set to 15.0
- Over the course of 40.0 frames, `height` is changed via a lerp to the y output of a BeizierCurve3 (that goes for the first 36.0 frames) from startp to startp + 0.34 in y with a ymax of 8.0
- `AcolyteVine`'s `basestate` and animstate set to 101
- `height` set to 3.4
- ShakeSprite called with intensity (0.1, 0.1, 0.0) with 0.2 frametimer
- `AcolyteVine`'s spin` zeroed out
- `AcolyteVine`'s `sprite` angles set to FlipAngle
- [position](../../Actors%20states/BattlePosition.md) changed to `Flying`
- `AcolyteVine` animstate set to its `basestate`
- `AcolyteVine` has the `BattleIdlef` animationClip played
- y `cursoroffset` decreased by 1.0
- `poisonres`, `freezeres`, `numbres` and `sleepres` set to 110 which makes this enemy immune to [Poison](../../Actors%20states/BattleCondition/Poison.md), [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md)

## Move 3 - Vine attack
A single target vine attack that may hit multiple times.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 1 time (from 1 to 2 times instead if hardmode us true, but the second hit requires the targer's `hp` to still be above 0)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (same target for each calls)|<ul><li>If harmode is false, 3</li><li>If hardmode is true, 2 on the first hit, 1 on the second hit</li></ul>|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- Camera moves to look near the midpoint of this enemy and `playertargetentity`
- animstate set to 103
- Yield for 0.6 seconds
- animstate set to 101
- Yield for 0.3 seconds
- `checkingdead` is set to a new VineAttack coroutine with 3 damageamt (2 instead if hardmode is true), actionid as the callerid without gettarget. The coroutine will set `checkingdead` to null upon completion
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
    - Over the course of 10.0 frames, the `Prefabs/Objects/VenusRoot` position changes to its position + the matching heading direction vector * 5.0. The first time reaching the 5th frame or later when `playerdata[playertargetID]`'s `hp` is above 0, DoDamage 1 call happens before the frame yield
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

## Move 4 - Aerial strike
A single target aerial strike that also changes the [position](../../Actors%20states/BattlePosition.md) to `Ground`.

### About status resistance
This move will undo any resitance increases of [Poison](../../Actors%20states/BattleCondition/Poison.md), [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md) conditions

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- y position increased by `height`
- `height` set to 0.0
- `sprite` local position set to Vector3.zero
- Camera moves to look near this enemy
- animstate set to 0 (`Idle`)
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.3 seconds
- animstate set to 108
- Yield for 0.2 seconds
- animstate set to 2 (`Jump`)
- `spin` set to (0.0, 15.0, -0.1)
- The `enemydata` index of the `AcolyteVine` is obtained (it should always be present from here)
- `AcolyteVine` animstate set to 18 (`KO`)
- `AcolyteVine`'s `hp` set to 0
- Camera moves to look near `playertargetentity`
- `ChargeDown4` sound plays
- `Spin2` sound plays on loop with 1.1 pitch and 0.6 volume
- `AcolyteVine` MainManager.[MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) down by 5.0 in y with 50.0 multiplier with smooth and without local
- Over the course of 40.0 frames, position is lerped to the midpoint between startp and `playertargetentity` + (0.75, 5.5, 0.0)
- Yield for 0.2 seconds
- `Spin2` sound stopped
- `Toss4` sound plays
- `spin` zeroed out
- `flip` set to false
- `sprite` local position set to Vector3.zero
- animstate set to 107
- `trail` set to true
- Over the course of 20.0 frames, position is lerped to `playertargetentity` + 1.0 in y
- `trail` set to false
- DoDamage 1 call happens
- ShakeScreen called with an ammount of (0.1, 0.1, 0.1) and 0.1 time
- `HugeHit` sound plays
- Yield for 0.05 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 110
- z `spin` set to -20.0
- Over the course of 40.0 frames, this enemy moves to startp + Vector3.up via a BeizierCurve3 with a ymax of 7.0 and `AcolyteVine` has its `startscale` and its `rotater` angles lerped to Vector3.zero
- position set to startp
- animstate set to 108
- `spin` zeroed out
- Yield a frame
- `sprite` angles reset to Vector3.zero
- [position](../../Actors%20states/BattlePosition.md) set to `Ground`
- Yield for 0.5 seconds
- `cursoroffset`, `poisonres`, `freezeres`, `numbres` and `sleepres` are reset to their default value from [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) without createentity
- `AcolyteVine` is destroyed
- [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called

## Move 5 - Phase transition
Transition to the second phase which spawns a new [AngryPlant](AngryPlant.md) enemy and performs move 2 (vine spawn).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true, but it doesn't affect anything for this move.

### Logic sequence

- `data[1]` set to 1
- `cantmove` decremented. This gives another actor turn to this enemy which will be move 4 (aerial strike) because [position](../../Actors%20states/BattlePosition.md) will always end up being `Flying` at the end of this actor turn
- Camera moves to look near this enemy
- animstate set to 103
- Yield for 0.6 seconds
- animstate set to 105
- Yield for 0.35 seconds
- Camera moves to the left by 3.0
- ShakeScreen called for 0.5 time
- Yield for 0.5 seconds
- `VenusBudAppear` sound plays
- [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add a new [AngryPlant](AngryPlant.md) enemy at this enemy position + (-3.0, 0.0, -0.1)
- DeathSmoke particles plays at the `AngryPlant`
- `AngryPlant`'s `startscale` set to Vector3.zero
- Yield a frame
- `AngryPlant` animstate set to 101
- `AngryPlant`'s y `spin` set to 20.0
- Over the course of 30.0 frames, `AngryPlant`'s `startscale` is lerped to (1.15, 1.15, 1.15)
- `AngryPlant`'s `startscale` set  to (1.15, 1.15, 1.15)
- `AngryPlant`'s `spin` zeroed out
- Yield for 0.5 seconds
- `AngryPlant`'s animstate set to 0 (`Idle`)
- `data[1]` set to 2 (completed the phase transition)
- `data[0]` set to 0 (forces the vine spawn move to be available for usage)
- Move 2 (vine spawn) is used
