# `HornDash`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|170.0|[TappingKey](../../Action%20commands/TappingKey.md)|{4.0, 8.0, 1.25, 1.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done for each enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground`|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` * `barfill` (value after DoCommand 1) ceiled then clamped from 1 to 99. If `barfill` is exactly 1.0 (the bar was fully filled), 1 is added after the clamp. NOTE: This clamp incorrectly removes the influence of negative `atk` values|[Flip](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|

## Logic sequence

- attacker MoveTowards in fron of the player party, but still to the left of the stage at 2.0 speed stopping at animstate 116
- The camera moves up and back slowly
- Yield until the attacker's `forcemove` is done
- attacker `overridejump` set to true
- attacker `overrideanim` set to true
- `Prefabs/Particles/WalkDust` particles instantiated childed to the attacker's `sprite` slightly up and back from it with x angle of -90.0
- Yield for a frame
- DoCommand 1 call happens
- Yield until `doingaction` goes to false (during the yield, the `Prefabs/Particles/WalkDust` particles's emission.rateOverTime is set to `barfill` * 20.0 clamped from 3.0 to 15.0 and their local position set to (Sin(Time.time * 7.0) * 0.25 * `barfill`, 0.2, -0.1) meaning they will oscillate left and right as `barfill` goes higher)
- Camera is reset to default
- attacker `rigid` has its gravity disabled
- `enemybounce` initialised to a new array with length being the `availabletargets`'s length
- attacker animstate set to 117
- `Prefabs/Particles/WalkDust` particles's main.startLifetime set to 1.5 and startSize to 1.25
- If `barfill` is higher than 0.3 (more than 30% filled), attacker's `trail` is set to true
- `FastWoosh` sound plays
- Over the course of x amount of frames where x is 1 / (0.04 * (`barfill` clamped from 0.45 to 1.0)) (basically, 25.0 frames if `barfill` is 1.0, 55.5556 frames if `barfill` is 0.45 or below), attacker moves from (-1.0, 0.0, -0.25) to (15.0, 0.0, 0.0). If they get past an enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` and they didn't received damages already:
    - DoDamage 1 call happens on the enemy party member
    - A BounceEnemy coroutine starts on the enemy party member and stored in `enemybounce` with the corresponding `availabletargets` index with 2 bounced and 1.5 multiplier
- Yield until all `enemybounce` elements goes to null (meaning all EnemyBounce coroutines are fully done)
- attacker's `trail` is set to false
- attacker position set to (-10.0, 0.0, 0.0)
- attacker `overridejump` is set to false
- attacker `overrideanim` is set to false
- attacker animstate set to 1 (`Walk`) with a SetAnimForce call
- `Prefabs/Particles/WalkDust` particles is destroyed
- attacker MoveTowards its initial position before this action
- Yield until attacker's `forcemove` is done
- attacker `rigid` gets its gravity enabled back
