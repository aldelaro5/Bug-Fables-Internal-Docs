# `Bee`'s basic attack

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|70.0|[RandomPressBar](../../Action%20commands/RandomPressBar.md)|{25.0, 70.0, 25.0, 2.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` unclamped|null|null|false|
|1_b|DoCommand 1 set `commandsuccess` to false|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` clamped from -99 to 1|null|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|

## Logic sequence

- If in `demomode`, CreateHelpBox is called with id 0
- `Bee` MoveTowards in front of party, but still left side of the stage at 2.0 multiplier
- Move the camera to be the halfway point between `Bee`'s destination position and targetentity (the enemy party member chosen)
- Yield until `Bee` is done with its `forcemove`
- `Bee`.animstate set to 103
- 0.17 seconds yield
- `Bee`.animstate set to 104
- DoCommand 1 call
- Frame yield
- Yield until `doingaction` is false (DoCommand is done)
- If `commandsuccess` is true (successful action command), `bee`.animstate is set to 105. If it's false (failed action command), it's set to 106 instead
- 0.05 seconds yield
- `Woosh` sound played on `sounds[8]` at 1.1 pitch on loop
- A `Prefabs/Objects/BeerangBattle` is instantiated and positioned above `Bee`. It will travel towards a position above `availabletargets[target]` (the selected enemy) and back using a BeizierCurve3 over the corse of 40.0 frames while continuously rotating on the z axis. The midpoint of the curve changes depending on `commandsuccess` (higher in the air if true, extruding the stage if false). The first time the object reaches the halfway point at 20.0 frames, its curve midpoint will change to be intruding the stage. On top of this, if `commandsuccess` is true when this happens, DoDomage 1_a is called
- The `BeerangBattle` is destroyed
- The `Woosh` sound is stopped with 0.1 fade speed
- if `commandsuccess` is false, DoDamage 1_b is called
- 0.15 seconds yield
- `bee`.animstate set to 13 (`BattleIdle`)
- 0.25 seconds yield
