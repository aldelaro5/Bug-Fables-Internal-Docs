# `HurricaneBeemerang`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|<ul><li>220.0 if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is not equipped</li><li>250.0 if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped</li></ul>|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{-1f, 6.5f, 0.15f, 1f, 0f, 0f, 0f, 0f, 0f, 4f} if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is not equipped</li><li>{-1f, 6.5f, 0.15f, 1f, 0f, 0f, 0f, 0f, 0f, 5f} if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped</li></ul>|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|`playerdata[currentturn]`|`availabletargets[target]`|`playerdata[currentturn].atk` - 1 clamped from 0 to 99. NOTE: This incorrectly ignores a base `atk` of 0 or lower due to the lower bound clamp|[Flip](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|2_a|DoDamage 1's result was higher than 0|null|`availabletargets[target]`|DoDamage 1's result - `i` clamped from 1 to 99, done `x` - 1 times where `i` starts at 1 and increments after each iteration and `x` is the amount of hits which is 2 + the amount of 1/2 parts contained in `barfill` value after DoCommand 1 (amount of 1/3 parts instead if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped)|Empty array|false|
|2_b|DoDamage 1's result was 0 or lower|null|`availabletargets[target]`|0, done `x` - 1 times where `x` is the amount of hits which is 2 + the amount of 1/2 parts contained in `barfill` value after DoCommand 1 (amount of 1/3 parts instead if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped)|null|null|false|

## Logic sequence

- `combo` set to 0
- Camera moved to zoom in slowly towards the left of the stage
- attacker MoveTowards in front of the party, but still towards the left side of the stage
- Yield until attacker is done with its `forcemove`
- attacker animstate set to 100
- Yield for 0.3 seconds
- Yield for a frame
- `Prefabs/Particles/Hurricane` is instantiated childed to the attacker's `sprite` slightly up and forward from it rotated -90.0 in x
- Yield for a frame
- `Woosh` sound played on `sounds[9]` at 0.5 pitch and 0.6 volume on loop
- DoCommand 1 happens
- Yield for a frame
- Yield until `doingaction` is false. Before each frame yield, `Prefabs/Particles/Hurricane`'s particle emission's RateOverTime is set to 40.0 * `barfill` clamped from 3.0 to 40.0 followed by attacker's y `spin` is set to 40.0 * (`barfill` clamped from 0.3 to 1) followed by `sounds[9].pitch` changed to `successfulchain` / 4.0 * 1.25 clamped from 0.5 to 1.5. NOTE: This pitch should have been based on `barfill` as `successfulchain` is not used for a [TappingKey](../../Action%20commands/TappingKey.md) action command. This means the pitch won't change before each frame yield and the exact pitch is dependant on the result of the last `TappingKey` action command which is unexpected
- `sounds[9]` is stopped at 0.1 speed
- `Toss` sound is played
- `Woosh` sound played on `sounds[8]` at 0.9 pitch and 0.5 volume on loop
- attacker's `spin` is zeroed out
- attacker animstate set to 101
- `Prefabs/Objects/BeerangBattle` instantiated at the attacker's position
- Camera reset to default
- `Prefabs/Objects/BeerangBattle` is set to go towards slightly up from the targetentity using a BeizierCurve3 with a midpoint being the actual midpoint of the entire trajectory, but a bit forward over the course of 20.0 frames with a z rotation of 20.0 * framestep each frame
- The amount of hits is calculated which is done by doing the following:
    - Take 2.0 * `barfill` (3.0 * `barfill` instead if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped)
    - Ceil the result and clamp it from 1 to 99
    - Add 1 (Add 2 instead if `barfill` is 1.0 or higher)
- `enemybounce` is set to a new array of one element being either null if targetentity's [deathtype](../../Actors%20states/Enemy%20features.md#deathtype) is 12 or if it isn't, a new BounceEnemy coroutine starting on targetentity. In practice, it means all but the `KeyR`, `KeyL`, `Tablet` [enemies](../../../Enums%20and%20IDs/Enemies.md#enemies) will have a BounceEnemy coroutine started on them if they are the target of this action
- `Prefabs/Particles/Hurricane`'s particle emission's RateOverTime is set to 50.0 * `barfill` clamped from 20.0 to 50.0
- `Prefabs/Objects/BeerangBattle` has a SpinAround added to it with a z `itself` of 35.0
- `HurricaneBig` particles are played without destroy at the targetentity position
- `combo` is set to 0
- For each amount of hits + 1:
    - If this is the first iteration:
        - DoDamage 1 call happens
        - `lastdamage` is set to a local int
    - Otherwise, if the `lastdamage` value of DoDamage 1 was higher than 0:
        - DoDamage 2_a call happens (index is this loop's index)
        - [ShowComboMessage](../../Visual%20rendering/ShowSuccessWord.md#showcombomessage) is called with targetentity without block. NOTE: This is incorrect because at this point, `combo` will be 2 which means the word 1 (GREAT) will be skipped
    - Otherwise:
        - DoDamage 2_b call happens
    - Yield for 0.75 / (amount of hits) seconds
- The `HurricaneBig` particles are placed offscreen then destroyed in 3.0 seconds
- The `Prefabs/Objects/BeerangBattle`'s SpinAround is destroyed
- The `Prefabs/Particles/Hurricane` is moved offscreen, rooted and destroyed in 3.0 seconds
- `Prefabs/Objects/BeerangBattle` is set to go towards the attacker using a BeizierCurve3 with a with a midpoint being the actual midpoint of the entire trajectory, but a bit forward over the course of 20.0 frames with a z rotation of 20.0 each frame
- `sounds[8]` is stopped
- `Prefabs/Objects/BeerangBattle` is destroyed
- attacker animstate set to 13 (`BattleIdle`)
- Yield for a frame if the first `enemybounce` isn't null
- Yield for 0.25 seconds
- targetentity basestate set to the one it had at the start of this action
