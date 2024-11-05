# `IceBeemerang`
This is a [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) with `Bee` and `Moth` and as such, it is assumed that the attacker is either one and it also means that [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) won't consume their actor turns like normal.

> NOTE: This action is unique in the sense that it is the only [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) that doesn't involve [GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) and instead uses the regular damage pipeline directly. This creates unique circumstances in which some issues arises in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) where the user of the move and the attacker sent to [DoDamage](../../Damage%20pipeline/DoDamage.md) may be different and lead to unexpected behaviors. Also, this action is excluded to benefit from the `Magic` weakness bonus in CalculateBaseDamage. For more information, check the CalculateBaseDamage documentation. 

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|2|DoCommand 1 set `commandsuccess` to true|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|3|DoCommand 2 set `commandsuccess` to true|54.5454521|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|4|DoCommand 3 set `commandsuccess` to true|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|5|DoCommand 4 set `commandsuccess` to true|54.5454521|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|6|DoCommand 5 set `commandsuccess` to true|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|7|DoCommand 6 set `commandsuccess` to true|54.5454521|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|8|DoCommand 7 set `commandsuccess` to true|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|9|DoCommand 8 set `commandsuccess` to true|54.5454521|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|10|DoCommand 9 set `commandsuccess` to true|47.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|11|DoCommand 10 set `commandsuccess` to true|42.727272|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|
|12|DoCommand 11 set `commandsuccess` to true|34|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

> NOTE: The clampings done in the damageammount columns can incorrectly remove the influence of negative `atk` value or disproportionally benefits positive [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) effects as opposed to a higher base `atk` value. Also note: the / 1.15 ceiled operation effectively doesn't change an `atk` value from 0 to 7, but it will it increase by 1 for a value of -2 or -1 and decrease it by 1 for a value of 8 or 9.

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|DoCommand 1 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|2|DoCommand 2 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled clamped from 0 to 99|null|empty array|false|
|3|DoCommand 3 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled - 1 clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|4|DoCommand 4 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled - 1 clamped from 0 to 99|null|empty array|false|
|5|DoCommand 5 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled - 2 clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|6|DoCommand 6 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled - 2 clamped from 0 to 99|null|empty array|false|
|7|DoCommand 7 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled - 3 clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|8|DoCommand 8 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled - 3 clamped from 0 to 99|null|empty array|false|
|9|DoCommand 9 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled - 1 clamped from 1 to 99 - 4 clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|10|DoCommand 10 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled - 1 clamped from 1 to 99 - 4 clamped from 0 to 99|null|empty array|false|
|11|DoCommand 11 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` / 1.15 ceiled - 2 clamped from 1 to 99 - 5 clamped from 0 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|12|DoCommand 12 set `commandsuccess` to true|`Bee`'s player party member|`availabletargets[target]`|`Bee`'s `atk` / 1.15 ceiled - 2 clamped from 1 to 99 - 5 clamped from 0 to 99|null|empty array|false|
|F1|DoCommand 1<sup>*</sup>, 2, 4, 6, 8, 10 or 12 set `commandsuccess` to false|`Bee`'s player party member|`availabletargets[target]`|1|null|empty array|false|
|F2|DoCommand 3, 5, 7, 9 or 11 set `commandsuccess` to false|`Moth`'s player party member|`availabletargets[target]`|1|null|empty array|false|

*: It is unexpected for F1 to happen instead of F2 if DoCommand 1 set `commandsuccess` to false because `Moth` would have attacked first, not `Bee`

## Custom `charge` reset logic

- After DoDamage 1 (if it occurs), `Moth`'s `charge` is set to 0. The same happen with DoDamage 3, 5, 7, 9 or 11, but they don't influence the logic since the value was already set to 0
- After DoDamage 2 (if it occurs), `Bee`'s `charge` is set to 0. The same happen after DoDamage 4, 6, 8, 10 or 12, but they don't influence the logic since the value was already set to 0

## Logic sequence

- Camera moves +1.5 in x towards the left side of the stage
- The `atk` of `Bee` / 1.15 ceiled and `Moth` / 1.15 ceiled are saved in a local array which will be refered to as their "rounded base atk" from now on
- Both `Bee` and `Moth` have [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them with `overrideflip` set to true
- `Bee` MoveTowards (-2.2, 0.0, -0.9) at 2.0x multiplier with a stopstate of 13 (`BattleIdle`)
- `Moth` MoveTowards (-3.65, 0.0, -1.55) at 2.0x multiplier with a stopstate of 13 (`BattleIdle`)
- Yield until `Bee` and `Moth`'s `forcemove` are done
- `Bee` animstate set to 0 (`Idle`)
- DoCommand 1 happens
- Yield until `doingaction` goes to false
- `Prefabs/Objects/BeerangBattle` is instantiated rooted
- If `commandsuccess` is false, refer to the [section below](#docommand-1-2-4-6-8-10-or-12-set-commandsuccess-to-false) when DoCommand 1 fails which will eventually end the action
- Otherwise, the following is done up to 6 times (index tracked in reverse so it goes from 6 to 1) as long as DoCommand calls keep setting `commandsuccess` to true (the first one that sets it to false causes a breakout early which will eventually end the action early):
    - If theres less than 3 cycles left:
        - The rounded base atk of `Bee` and `Moth` decreases by 1 and then clamped from 1 to 99
        - FinishAction is called manually (shouldn't do anything in practice)
        - Yield for a frame
    - `Prefabs/Objects/BeerangBattle` moved offscreen
    - `Bee` animstate set to 103
    - `Moth` animstate set to 111
    - `mothicenormal` particles plays at the world [CenterPos](../../Actors%20states/CenterPos.md) of `target` enemy party member - Vector3.up
    - `IceMothHit` sound plays
    - DoDamage 1 + 2x call happens (`Moth` as attacker) where x starts at 0 and increments after each cycle
    - `Moth`'s `charge` set to 0
    - Yield for 0.5 - `combo` * 0.025 clamped from 0.0 to 1.0 seconds (this wait gets shorter as more DoDamage call happens within this action)
    - `Moth` animstate set to 102
    - Yield for 0.1 seconds
    - DoCommand 2 + 2x call happens where x starts at 0 and increments after each cycle
    - Yield for a frame
    - Yield until `doingaction` goes to false
    - `Toss` sound plays
    - If `commandsuccess` is false, refer to the [section below](#docommand-1-2-4-6-8-10-or-12-set-commandsuccess-to-false) where `Bee`'s associated DoCommand fails which will breakout of this loop early and eventually end this action
    - Yield for 0.1 seconds
    - `Toss2` sound plays on `sounds[9]` with pitch 1.0 + 0.1 * `combo` on loop (this sound will increase pitch after each DoDamage calls within this action)
    - `Bee` animstate set to 105
    - Over the course of the last DoCammand's `timer` value / 1.5 + 1.0 frames, `Prefabs/Objects/BeerangBattle` moves from `Bee`'s position + 1.0 in y to the world [CenterPos](../../Actors%20states/CenterPos.md) of `target` enemy party member. During the move, the object z angle is set to framestep * 10.0 + 2.0 * `combo` (meaning it rotates faster after each DoDamage call within this action)
    - DoDamage 2 + 2x call happens (`Bee` as attacker) where x starts at 0 and increments after each cycle
    - `Bee`'s `charge` set to 0
    - FinishAction is called manually (shouldn't do anything in practice)
    - Yield for 0.1 seconds
    - If this isn't the last cycle, DoCommand 3 + 2x call happens where x starts at 0 and increments after each cycle
    - Over the course of the first DoCammand call of this cycle's `timer` value + 1.0 frames, `Prefabs/Objects/BeerangBattle` moves from the world [CenterPos](../../Actors%20states/CenterPos.md) of `target` enemy party member to `Bee`'s position + 1.0 in y via a BeizierCurve3 with a ymax of 8.0. During the move, the object z angle is set to framestep * -10.0 - 2.0 * `combo` (meaning it rotates faster after each DoDamage call within this action)
    - `sounds[9]` is stopped
    - Yield until `doingaction` goes to false
    - If this is the last cycle, `Bee` animstate set to 13 (`BattleIdle`). Otheriwse, `mothicenormal` particles plays at the world [CenterPos](../../Actors%20states/CenterPos.md) of `target` enemy party member - 1.0 in y (uniform scale of 0.5x if `commandsuccess` is false)
    - If `commandsuccess` is false refer to the [section below](#docommand-3-5-7-9-or-11-set-commandsuccess-to-false) where `Moth`'s associated DoCommand fails which will breakout of this loop early and eventually end this action
    - If `target` enemy party member's `hp` reached 0 or below:
        - `commandsuccess` is set to true
        - `target` enemy party member's `flip` is set to false
        - Refer to the [ending section](#ending) below where the action will be ended early
    - FinishAction is called manually (shouldn't do anything in practice)
    - Yield for a frame
- Once all cycles are done without a breakout early:
    - `target` enemy party member's `flip` is set to false
    - Refer to the [ending section](#ending) below where the action will be ended early

### DoCommand 1, 2, 4, 6, 8, 10 or 12 set `commandsuccess` to false
This happens whenever a `Bee` related action command is failed and it causes the action to end early:

- FinishAction is called manually (shouldn't do anything in practice)
- `Bee` animstate set to 106
- `Moth` animstate set to 14 (`Sleep`)
- `Toss2` sound plays on `sounds[9]` with pitch 0.9 on loop
- `Prefabs/Objects/BeerangBattle` angles zeroed out
- Over the course of 46.0 frames, `Prefabs/Objects/BeerangBattle` moves from `Bee`'s position + 1.0 in y to the world [CenterPos](../../Actors%20states/CenterPos.md) of `target` enemy party member via a BeizierCurve3 with a ymax of 8.0. During the move, the object's z angle is set to framestep * 10.0
- `sounds[9]` is stopped
- `AtkFail` sound plays
- DoDamage F1 call happens
- `target` enemy party member's `flip` is set to false
- Refer to the [ending section](#ending) below where the action will be ended early

### DoCommand 3, 5, 7, 9 or 11 set `commandsuccess` to false
This happens whenever a `Moth` related action command is failed and it causes the action to end early:

- `Prefabs/Objects/BeerangBattle` is moved offscreen
- `IceMothMiss` sound plays
- DoDamage F2 call happens
- `Bee` animstate set to 11 (`Hurt`)
- `Moth` animstate set to 110
- Yield for 1.0 seconds
- `IceShatter` particles plays at `Moth`'s position + 1.0 in y
- `Moth` animstate set to 13 (`BattleIdle`)
- Refer to the [ending section](#ending) below where the action will be ended early

### Ending
All branches ends here eventually:

- `Prefabs/Objects/BeerangBattle` is destroyed
- Yield for 0.5 seconds
- Camera resets to default
- Both `Bee` and `Moth` MoveTowards their initial position before this action with 2.0x speed with `overrideflip` set to true
- Yield until `Bee` and `Moth`'s `forcemove` are done
- Both `Bee` and `Moth`'s positions are set to their initial position before this action with `overrideflip` set to false and `flip` set to true
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) is called with 0 and 2 (normally `Bee` and `Moth`)
