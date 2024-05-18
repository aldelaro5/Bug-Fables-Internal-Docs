# `Moth`'s basic attack

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|25.0|[RandomPressKeyTimer](../../Action%20commands/RandomPressKeyTimer.md)|{0.0, 4.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` unclamped|[Magic](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|1_b|DoCommand 1 set `commandsuccess` to false|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` clamped from 0 to 1. NOTE: This incorrectly removes influences of negative `atk` values|null|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|

## Logic sequence
- If in `demomode`, CreateHelpBox is called with id 2
- Camera is zoomed in at 0.01 speed from the midpoint between `Moth` and targetentity
- `IceMothThrow` sound plays
- `Moth` animstate set to 108
- A `Prefabs/AnimSpecific/mothbattlesphere` is instantiated a bit up-left from `Moth` position with 90.0 degress rotatex in x with a `Prefabs/Objects/SingleSphere` as child at half transparency. The `mothbattlesphere` will be destroyed in 2.0 seconds
- DoCommand 1 happens
- Yield until DoCommand is done
- Camera is reset to default at 0.3 speed
- A `Prefabs/Particles/mothicenormal` appears slighly up from targetentity with x rotation of -90.0 and destroyed in 2.0 seconds
- `Prefabs/Objects/icepillar` appears at targetentity with x rotation of -90.0, a DialogueAnim with x/y targetscale of 0.5 and destroyed in 4.0 seconds
- If `commandsuccess` is true (action command succeeded):
    - `Moth` animstate set to 111
    - DoDamage 1_a happens
    - `IceMothHit` sound plays
    - 1.0 second yield
- Otherwise (action command failed):
    - `icepillar`'s DialogueAnim targetscale is set to (0.4, 0.4, 0.5)
    - `Moth` animstate set to 109
    - `IceMothMiss` sound plays
    - DoDamage 1_b happens
    - 1.2 seconds yield
    - A `Prefabs/Particles/IceShatter` appears slighly up from `Moth` with x rotation of -90.0 and destroyed in 1.0 seconds
    - `Moth` animstate set to 0 (`Idle`)
- If `Prefabs/Objects/icepillar` still exists, its DialogueAnim is set to shrink at 0.05 speed
