# `BeeFly`
This is a [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) with `Bee` and `Beetle` and as such, it is assumed that the attacker is either one and it also means that [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) won't consume their actor turns like normal.

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|170.0|[TappingKey](../../Action%20commands/TappingKey.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input), 8.0, 0.8, 1.0}|
|2|DoCommand 1 set `barfill` above 0.5|50.0 * `barfill` clamped from 25.0 to 50.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 2 executed and set `commandsuccess` to true|null|`availabletargets[target]`|[GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) for `Bee` and `Beetle` + 2 then 2 is added to it if `barfill` is 1.0 or higher|[Pierce](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
|1_b|Either DoCommand 2 didn't executed or it did, but set `commandsuccess` to false|null|`availabletargets[target]`|([GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) for `Bee` and `Beetle` + 2 then 2 is added to it if `barfill` is 1.0 or higher) * 0.25 floored clamped from 1 to 99|null|empty array|false|
|2|DoDamage 1_a happened and the `target` enemy party member [position](../../Actors%20states/BattlePosition.md) is `Flying` with a `ToppleFirst` [weakness](../../Actors%20states/Enemy%20features.md#weakness)|null|`availabletargets[target]`|1|[Pierce](../../Damage%20pipeline/AttackProperty.md)|{[NoCounter](../../Damage%20pipeline/DoDamage.md#nocounter)}|false|

## Logic seqeunce

- Both `Bee` and `Beetle` have [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them with `overrideanim` and `overridejump` set to true
- `Jump` sound plays
- Over the course of 31.0 frames, `Bee` moves to `Beetle` + Vector3.up via a BeizierCurve3 with a ymax of 4.0. During the move, `Bee` animstate is set to 2 (`Jump`)
- Camera moves slowly to look at the `target` enemy party member + 4.25 + its `height` in y and -9.5 in z
- `BeeFly` sound plays on `sounds[9]` on loop
- `Bee` position set to `Beetle`'s - 0.1 in z
- `Bee` and `Beetle` animstate set to 102
- `Beetle` gets childed to `Bee`
- DoCommand 1 call happens and it's set to `actionroutine`
- Over the course of 171.0 frames, `Bee` moves towards the `target` enemy party member's world [CenterPos](../../Actors%20states/CenterPos.md) + (0.0, 0.0 -0.1) in x and for the y, it's the same, but + 5.0 + a lerp from 0.5 to 5.0 with a factor of `barfill` (the z is 0.0). Essentially, `Bee` will move up more as `barfil` fills up, but will still align themselves with the `target` enemy party member horizontally. During the move, `sounds[9]` pitch is set to a lerp from 1.0 to 1.2
- `sounds[9]` is stopped
- `Fall` sound plays
- `Beetle` gets childed back to the `battlemap`
- `Beetle` animstate set to 119
- `Beetle`'s `sprite`.flipY set to true
- `Beetle`'s y `spin` set to 25.0 * `barfill`
- `Bee` animstate set to 114
- If `barfill` is higher than 0.5 (by then, DoCommand 1 should be done even if it wasn't confirmed finished):
    - FinishAction is called manualled to end DoCommand 1 if it wasn't already (but it should be by then)
    - Yield for 0.5 seconds
    - DoCommand 2 call happens
- Over the course of x frames, but continued indefinetely if `doingaction` is still true (x is 50.0 * `barfill` clamped from 25.0 to 50.0):
    - `Bee` moves to 20.0 in x and up by 10.0 in y from their current position via a BeizierCurve3 with a ymax of -3.0
    - `Beetle` moves to the `target` enemy party member's world [CenterPos](../../Actors%20states/CenterPos.md) + (0.0, 0.0 -0.1)
- `Bee` is moved offscreen
- If `commandsuccess` is true (by this point, DoCommand 2 is confirmed finished):
    - `AtkSuccess` sound plays
    - ShakeScreen with 0.2 ammount and 0.5 time with dontreset
    - `HugeHit` sound plays
    - DoDamage 1_a call happens
    - If the `target` enemy party member [position](../../Actors%20states/BattlePosition.md) is `Flying` with a `ToppleFirst` [weakness](../../Actors%20states/Enemy%20features.md#weakness), DoDamage 2 call happens
    - Yield until `Beetle` y position gets to 0.0 or lower while it decreases by 1/4 of the game's frametime every frame
    - `DirtExplode` particles plays at `Beetle` position
    - `Dig` sound plays at 1.2 pitch
    - `Beetle`'s `digging` is set to true
    - `Beetle`'s `sprite`.flipY is reset to false
    - `Beetle`'s `spin` is zeroed out
    - Yield for 0.5 seconds
- Otherwise (`commandsuccess` is false):
    - DoDamage 1_b call happens
    - `Beetle` animstate set to 105
    - `Beetle`'s `sprite`.flipY is reset to false
    - `Beetle`'s `spin` is zeroed out
    - Over the course of 51.0 frames, `Beetle` moves to its initial position before this action via a BeizierCurve3 with a ymax of 5.0
- Camera is reset to default
- Over the course of 46.0 frames:
    - `Bee` moves to their initial position before this action + Vector3.down
    - If `commandsuccess` is true, `Beetle` moves to its initial position before this action
- `Bee` animstate set to 13 (`BattleIdle`)
- `Bee` has [SetAnim](../../../Entities/EntityControl/Animations/SetAnim.md) called with empty args with force
- `Bee` position is set to their initial position before this action
- If `Beetle` is `digging`:
    - `DigPop` sound plays
    - `DirtExplode` particles plays at `Beetle` position
- Both `Bee` and `Beetle` have [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them with `digging`, `overrideanim` and `overridejump` set to true
- [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) is called with 0 and 1 (normally `Bee` and `Beetle`)
