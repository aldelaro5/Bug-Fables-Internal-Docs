# `PebbleTossPlus`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|180.0|[SequentialKeys](../../Action%20commands/SequentialKeys.md)|{5.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1a|DoCommand 1 set `combo` to 1 or above|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` + (`combo` - 2)|null|Empty array|false|
|1b|DoCommand 1 set `combo` to less than 1|null|`playerdata[currentturn]`|3 (later, the attacker's `hp` gets clamped from 1 to its `maxhp` so this cannot be lethal)|null|null|false|
|2|DoCommand 1 set `combo` to 3 or above, done up to 2 times for each of the enemy party members directly adjacent to the target of DoDamage 1a whose [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach`|null|The enemy party member|2|null|null|false|

## Multi targets
This action can target the enemy party member directly to the left or right of the original target if they exists and their [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach`.

## Logic sequence

- Camera moves slowly to the left of the stage slightly up and back
- CreateRock is created at half uniform scale offscreen under the map and assigned to a local called boulder
- attacker MoveTowards the back of the party at 2.0 speed
- Yield until attacker's `forcemove` is done
- `Dig` sound plays
- KabbuDig coroutine starts and set to `checkingdead` which animates the attacker's `sprite` going underground with `Prefabs/Particles/DirtFlying` particles and its `overrideanim`, `overrideflip` and `overrideheight` going to true
- Yield until `checkingdead` is done
- Yield for 0.5 seconds
- ShakeScreen for 0.2 ammount for 0.5 time
- attacker `overrideanim`, `overrideflip` and `overrideheight` set back to false
- attacker animstate set to 103
- `DirtExplode2` particles played at the attacker position
- `PebbleAtkDigUp` sound plays
- Over the course of 31.0 frames, the boulder moves via SmoothLerp towards (0.0, 12.0, 0.0). The first iteration after 15.5 frames will have the DoCommand 1 call happen (and set to `actionroutine`)
- Over the course of 204.0 frames, the boulder moves via Lerp from (0.0, 42.5, 0.0) to the attacker + Vector3.up. However, whenever `commandsuccess` goes to true or `actionroutine` is done, each frame may be counted multiple times which accelerates the movements (2x until 50% then 3x until 80% and then back to 1x)
- FinishAction is called manually which force ends the action (it normally will end before however because the `timer` is shorter then the full time it takes for the boulder to move)
- If `combo` is 0 or below:
    - `RockBreak` sound plays
    - RockBreak is called which instantiates a `Prefabs/Objects/CrackRockBreak` childed to the boulder and gets the boulder destroyed
    - ShakeScreen with 0.2 ammount for 0.5 time
    - DoDamage 1b call happens
    - attacker `hp` set to clamp from 1 to its `maxhp` (meaning DoDamage 1b cannot be lethal to the attacker)
- Otherwise (`combo` is 1 or higher):
    - Camera moves 2.0 in x
    - attacker animstate set to 104
    - `PebbleAtkLaunch` sound plays
    - Over the course of 41.0 frames, the boulder moves from (0.0, 2.0, 0.0) towards the target + its `height` + 1.0 in y using a `BeizierCurve3` with a ymax of the target's `height` + 5.0
    - `PebbleAtkHit1` sound plays
    - DoDamage 1a call happens
    - If `combo` is 3 or above:
        - `PebbleAtkHit2` sound is set to play in 0.5 seconds
        - 2 new boulders are instantiated with a uniform 0.25 scale at the original target + Vector3.up. One will go towards the left enemy party member positon + its `height` and the other will do the same, but for the right enemy party member. If no enemy party members exists there, it will instead go towards the attacker + 2.5 in x. The movement is done via ArcMovement for 30.0 frames at 4.0 height with no spin and with destroy after. If the enemy party member exists and its [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach`, DoDamage 2 call is set to happen in 0.5 seconds
    - RockBreak is called which instantiates a `Prefabs/Objects/CrackRockBreak` childed to the boulder and gets the boulder destroyed
    - Yield for 0.5 seconds if any directly adjacent enemy party member was hit by DoDamage 2, 
- Yield for 0.5 seconds
