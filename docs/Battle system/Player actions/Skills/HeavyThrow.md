# `HeavyThrow`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|235.0|[TappingKey](../../Action%20commands/TappingKey.md)|{-1.0, 6.0, 0.25, 1.0}
|2|None|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{random between 4 and 6 (Confirm, Cancel or Switch)}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` + (3.0 * `barfill` floored)|null|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|2|DoCommand 2 set `commandsuccess` to false|null|`playerdata[currentturn]`|3 (this can be lethal)|null|Empty array|false|

## Additional status infliction logic
After DoDamage 1 call, if DoCommand 1 set `barfill` to 0.85 or higher, [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called on the target to inflict a [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) for 2 turns with visual effects.

## Caveats with DoCommand 2
This action command is set to run for 60.0 frames, but the game won't actually wait that it is fully finished and leave up to 13 frames unaccounted for where the command is still running, but the game will read its result. This means that if the player succeeds the command, but was late enough, it will still result in DoDamage 2 call happening even if the command was succeeded. This means effectively, the action command's timing is 17.0 frames instead of 30.0 with the last 13.0 frames not being seen as a success by the action despite the UI indicating that it was.

# Logic sequence

- Camera moves slowly towards the left of the stage and back a bit up
- attacker MoveTowards in front of the player party at 2.0 speed
- Yield until attacker's `forcemove` is done
- attacker animstate set to 100
- Yield for 0.17 seconds
- DoCommand 1 call happens
- Yield until `doingaction` goes to false (during the yield, attacker's y `spin` goes from 20.0 to 40.0 with a factor of `barfill` and if `barfill` goes above 0.5, a ShakeSprite happens on the attacker for 0.1 intensity and 1.0 frametimer with an additional yield for a frame)
- attacker `sprite` local positon reset to Vector3.zero
- attacker animstate set to 101
- `Prefabs/Objects/BeerangBattle` instantiated without angles with a SpinAround that has an `itself` of (0.0, 0.0, 30.0 * `barfill` clamped from 0.5 to 1.0) 
- `Toss8` sound plays
- Over the course of x frames, `Prefabs/Objects/BeerangBattle` moves to the world [CenterPos](../../Actors%20states/CenterPos.md) of the target where x is 15.0 * lerp from 2.0 to 1.0 with a factor of `barfill` clamped from 0.5 to 1.0 (maximum is 15.0 frames, but it will go towards 7.5 frames as `barfil` gers lower)
- ShakeScreen with 0.4 * `barfill` ammount and 0.75 time
- DoDamage 1 call happens
- If `barfill` is 0.85 or higher, [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called on the target to inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) for 2 turns with visual effects
- `HugeHit` sound plays
- Over the course of 81.0 frames, `Prefabs/Objects/BeerangBattle` moves towards the attacker + Vector3.up via a BeizierCurve3 with a ymax of 15.0. On the first frame that's more than 40% of the way (33th frames and later), DoCommand 2 call happens
- `Prefabs/Objects/BeerangBattle` is destroyed
- If `commandsuccess` is false:
    - ShakeScreen with 0.1 ammount for 0.75 time
    - attacker `overrideanim` set to true
    - attacker animstate set to 11 (`Hurt`)
    - DoDamage 2 call happens
- Otherwise:
    - `DLGammaStep` sound plays
    - attacker animstate set to 100
- SlowSpinStop called on the attacker with (0.0, 30.0, 0.0) spinamount for 60.0 frametime
- Yield for 1.0 seconds
- The SlowSpinStop coroutine is stopped
- attacker `spin` is zeroed out
- attacker `overrideanim` set to false
- FlipAngle(true) called on the attacker which sets its y `sprite` angle to 0.0 or 180.0 according to its `flip`
