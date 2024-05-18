# `BeeRangMultiHit`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|0.01|[LongRandomBar](../../Action%20commands/LongRandomBar.md)|<ul><li>{3.0, 1.25} if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is not equipped</li><li>{4.0, 1.25} if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped</li></ul>

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|`playerdata[currentturn]`|`availabletargets[target]`|`playerdata[currentturn].atk` - 1 clamped from 0 to 99. NOTE: This incorrectly ignores a base `atk` of 0 or lower due to the lower bound clamp|null|Empty array|false|
|2_a|DoDamage 1's result was higher than 0|null|`availabletargets[target]`|DoDamage 1's result - `i`, done `successfulchain` - 1 times (value after DoCommand 1) where `i` starts at 1 and increments after each iteration|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|2_b|DoDamage 1's result was 0 or lower|null|`availabletargets[target]`|0, done `successfulchain` - 1 times (value after DoCommand 1)|null|null|false|

## Logic sequence

- `combo` set to 0
- Camera moved to zoom in slowly towards the left of the stage
- attacker MoveTowards in front of the party, but still towards the left side of the stage
- Yield until attacker is done with its `forcemove`
- attacker animstate set to 100
- Yield for 0.3 seconds
- DoCommand 1 happens
- Yield for a frame
- `Woosh` sound played on `sounds[9]` at 0.5 pitch and 0.6 volume on loop
- Yield until `doingaction` is false. Before each frame yield, attacker's y `spin` is set to 10 * (`successfulchain` + 1) followed by `sounds[9].pitch` changed to `successfulchain` / 4.0 * 1.25 clamped from 0.5 to 1.5
- `sounds[9]` is stopped at 0.1 speed
- `Toss` sound is played
- `Woosh` sound played on `sounds[8]` at 0.9 pitch and 0.5 volume on loop
- attacker's `spin` is zeroed out
- attacker animstate set to 101
- `Prefabs/Objects/BeerangBattle` instantiated at the attacker's position
- Camera reset to default
- If `successfulchain` higher than 0, `commandsuccess` is set to true
- For each `successfullchain` + 1:
    - `Prefabs/Objects/BeerangBattle` is set to go towards slightly up from the targetentity using a BeizierCurve3 with a midpoint up and back the target over the course of 40.0 frames (12.5 frames instead if it's not the first iteration) with a z rotation of 20.0 each frame
    - If this is the first iteration:
        - DoDamage 1 call happens
        - `lastdamage` is set to a local int
        - `combo` is decremented (undoes what DoDamage did so it stays at 0)
    - Othewise, if the `lastdamage` value of DoDamage 1 was higher than 0:
        - DoDamage 2_a call happens (index is this loop's index)
        - [ShowComboMessage](../../Visual%20rendering/ShowSuccessWord.md#showcombomessage) is called with targetentity without block
    - Otherwise:
        - DoDamage 2_b call happens
    - If this isn't the last iteration, `Prefabs/Objects/BeerangBattle` is set to go towards slightly up from its position using a BeizierCurve3 with a midpoint up and back forward its initial position over the course of 1 / 0.045 frames (~22.22 frames) with a z rotation of 20.0 each frame
    - `sounds[8].pitch` is increased by 0.07
- If `successfulchain` is 3 or higher, `AtkSuccess` sound plays
- `Prefabs/Objects/BeerangBattle` is set to go towards the attacker using a BeizierCurve3 with a ymax of 5.0 over the course of 40.0 frames with a z rotation of 20.0 each frame
- `sounds[8]` is stopped
- attacker animstate set to 13 (`BattleIdle`)
- `Prefabs/Objects/BeerangBattle` is destroyed
- If targetentity's `hp` is above 0, its animstate is set to its basestate
- Yield of 0.2 seconds
