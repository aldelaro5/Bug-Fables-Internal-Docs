# `Cape`

## [StartBattle](../../StartBattle.md) special logic involing `exp`
After this enemy is loaded in StartBattle, if all of the following is true, the `exp` is incremented by the floored result of a lerp from 10.0 to 3.0 with a factor of instance.`partylevel` / 27.0:

- battleentity.`forcefire` is true or we are in the `GiantLair` [area](../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../Enums%20and%20IDs/Maps.md)
- instance.`partylevel` is less than 27 (meaning it's not maxed)
- [flags](../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)

For more information, consult the [exp logic](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) documentation.

## [Fire](../../Actors%20states/BattleCondition/Fire.md) damage infliction logic in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This enemy has a 50% chance to be on fire on a `Fire` property attack if the current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`. It cannot be on fire otherwise.

## [GetEnemyPortrait](../../../TextAsset%20Data/Enemies%20data.md) logic
If [flag](../Flags%20arrays/flags.md) 664 is true (approached the oven during Chapter 7), the portrait sprite index field of this enemy is ignored and 225 is returned instead which includes the fire variant of this enemy in the portrait.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the cloth wrap move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the cloth wrap move, the damageammount of each DoDamage calls is decreased by 1
- In the freezing wind move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)

## Move selection
3 moves are possible:

1. A party wide aerial strike
2. A single target cloth wrap that continuously drains `hp`
3. A party wide freezing wind attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|2/5|
|2|2/5|
|3|1/5|

However, if move 3 is selected and either of the following is true, a continue directive is issued to the action loop which effectively rerolls the move selection:

- `forcefire` is true
- The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`

## Move 1 - Aerial strike
A party wide aerial strike.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Happens if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|This enemy|3|[Fire](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|
|2|Happens if DoDamage 1 didn't|This enemy|2|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 100
- `Toss7` sound plays
- Over the course of 61.0 frames, this enemy moves to its position + (2.0, 1.5, -0.1) via a SmoothLerp
- `Spin3` sound plays on loop
- animstate set to 104
- y `spin` set to 20.0
- Over the course of 41.0 frames this enemy moves to (-8.0, 1.0, 0.0) via a BeizierCurve3 with a ymax of -1.0. On the first frame that this enemy x position becomes lower than -4.5, PartyDamage 1 or 2 happens depending on which one has its conditions fufilled
- Yield for 0.5 seconds
- `Spin3` sound stopped
- `spin` zeroed out
- animstate set to 0 (`Idle`)
- Over the course of 41.0 frames, this enemy moves to startp via a SmoothLerp

## Move 2 - Cloth wrap
A single target cloth wrap that continuously drains `hp`.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|-1.0 (infinite)|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Called once if `barfill` is less than 1.0 (not full) during DoCommand 1 once 0.5 seconds passed and then continously called again every 1.25 seconds until either `barfill` reaches 1.0 (checked 0.75 seconds after each call) or that `playerdata[playertargetID]`'s `hp` becomes 0|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target is the same for each calls)|1 (0 instead if hardmode is true<sup>1</sup>). 1 is added to this if at least one of the following is true:<ul><li>`forcefire` is true</li><li>The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `GiantLair` while the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairFridgeInside`</li></ul>|[Atleast1pierce](../../Damage%20pipeline/AttackProperty.md)<sup>2</sup>|null|false|

1: This doesn't mean the call won't deal any damage because not only `Atleast1pierce` guarantees 1 damage is dealt, but also because it is meant to function with `hardatk` in mind which takes effect when hardmode is true
2: Enemy piercing damages are disabled, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `bobrange` and `bobspeed` set to 0.0
- Camera moves to look neat `playertargetentity`
- animstate set to 1 (`Walk`)
- `ClothAttack` sound plays
- Over the course of 61.0 frames, this enemy moves to `playertargetentity` position + (2.0, 1.5, -0.1) via a SmoothLerp
- animstate set to 100
- Yield for 0.5 seconds
- `playertargetentity` animstate set to 11 (`Hurt`)
- animstate set to 105
- `Woosh3` sound plays
- Over the course of 11.0 frames, this enemy moves to `playertargetentity` position - 0.1 in z via a lerp
- animstate set to 103
- DoCommand 1 call happens
- `infinitecommande` set to true which will prevent DoCommand 1 to end when MainManager.`mashcommandalt` is true until the action command is succeeded (it doesn't matter that this is set after the call because it will yield a frame in this case before reading the value)
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the [TappingKey](../../Action%20commands/TappingKey.md) command's help)
- This enemy positon is set to `playertargetentity` + (0.0, 0.0 - `height`, -0.1)
- As long as `playerdata[playertargetID]`'s `hp` is above 0 and `actionroutine` isn't null (meaning DoCommand 1 hasn't ended yet):
    - Yield for 0.5 seconds
    - If `barfill` is less than 1.0 (it isn't full yet):
        - DoDamage 1 call happens
        - If `lastdamage` is above 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy by `lastdamage` amount of `hp`
    - Yield for 0.75 seconds
- `Barfill` sound stops
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- If `playerdata[playertargetID]`'s `hp` is above 0, their animstate is set to the value they had before this action
- FinishAction is called manually to end DoCommand 1 if it wasn't already (but it should be by then)
- `bobrange` and `bobspeed` restored to their value before this action
- animstate set to 102
- Over the course of 61.0 frames, this enemy moves to `playertargetentity` position + (2.0, 1.5, -0.1) via a SmoothLerp
- animstate set to 0 (`Idle`)
- Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Over the course of 61.0 frames, this enemy moves to startp via a SmoothLerp
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called

## Move 3 - Freezing wind
A party wide freezing wind attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2|[Freeze](../../Damage%20pipeline/AttackProperty.md)|true if `barfill` is 1.0 or above after DoCommand 1 (meaning the bar got filled)<sup>1</sup>, false otherwise|0.0|Vector3.zero|false|null|

1: Not only the super block timing is affected by the [block timing issue](../../DoCommand.md#known-issues-with-getblock), `blockcooldown` is set to 0.0 before DoCommand 1 which removes all super block frames leading to the DoCommand 1 call. This only leaves the frame of the PartyDamage 1 call meaning in order to super block this, the player must press on the exact frame the PartyDamage call happens. It is a frame perfect window.

### Logic sequence

- animstate set to 1 (`Walk`)
- Over the course of 61.0 frames, this enemy moves to (1.0, 0.5, 0.0) via a SmoothLerp
- `checkingdead` set to a new WingAttack coroutine
- Yield all frames until `checkingdead` is null (the coroutine completed)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 0 (`Idle`)
- Over the course of 61.0 frames, this enemy moves to startp via a SmoothLerp

Here is what the WingAttack coroutine effectively ends up doing:

- `blockcooldown` is set to 0.0 (it effectively cancels any blocking leading to DoCommand 1 which leaves only the 1 frame that PartyDamage 1 is called as the super block timing)
- Camera moves to look near `partypos[0]` (the player party member in front)
- animstate set to 102
- `Woosh` sound plays on loop using `sounds[8]` with 0.5 pitch and 0.7 volume
- `Prefabs/Particles/CapeSmoke` GameObject instantiated rooted at (-4.0, 0.0, 0.0)
- DoCommand 1 call happens
- As long as DoCommand 1 is still ongoing and that it's been at least 80.0 frames, this is what happens on each frame:
    - For each player party members with an `hp` above 0, their y `spin` is set to (the frame index starting at 0.0 / 40.0 clamped from 0.0 to 1.0) * 20.0
    - `sounds[8]`'s pitch is set to the frame index starting at 0.0 / 80.0 clamped from 0.4 to 1.0
- `Prefabs/Particles/CapeSmoke` is moved offscreen at -9999.0 in y then destroyed in 1.0 seconds
- All player party members has their `spin` zeroed out
- PartyDamage 1 call happens
- Yield for 0.5 seconds
- `checkingdead` set to null which informs the caller that this coroutine completed
