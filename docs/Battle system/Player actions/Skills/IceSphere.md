# `IceSphere`
This is a [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) with `Bee`, `Beetle` and `Moth` and as such, it is assumed that the attacker is one of them and it also means that [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) won't consume their actor turns like normal.

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|275.0|[SequentialKeys](../../Action%20commands/SequentialKeys.md)|{6.0, 1.0}|
|2|None|53.3333321|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done for each enemy party member that fufills these conditions:<ul><li>Their [position](../../Actors%20states/BattlePosition.md) isn't `OutOfReach` or `Underground`</li><li>Their `height` is higher than 3.0</li><li>The iceball x position + 1.0 passes the enemy party member's x position (see the section below for details)</li></ul>|null|The enemy party member|[GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) with `Bee`, `Beetle` and `Moth` (halved ceiled if DoCommand 2 set `commandsuccess` to false) * (`barfill` if it's 0.9 or higher otherwise, take `barfill` * 0.8 clamped from 0.25 to 1.0) ceiled clamped from 2 to 99|[Freeze](../../Damage%20pipeline/AttackProperty.md)|empty array|false|

## [DamageWithinPos](../../Damage%20pipeline/DamageWithinPos.md) calls

|#|Conditions|playerid|damageammount|property|radius|xonly|
|-:|---|---|---|---|---|---|
|1|DoCommand 2 set `commandsuccess` to true and the Confirm input was pressed to explode the iceball. The iceball's position is effectively used as the position since `Bee`'s position is set to it (see the section below for details)|0 (`Bee`)|[GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) with `Bee`, `Beetle` and `Moth` (halved ceiled if DoCommand 2 set `commandsuccess` to false) / 2.0 * (`barfill` if it's 0.9 or higher otherwise, take `barfill` * 0.8 clamped from 0.25 to 1.0) clamped from 1.0 floored|[Freeze](../../Damage%20pipeline/AttackProperty.md)|3.0|false|

## Iceball rolling logic
The damaging logic of this action is affected by an iceball rolling to the right towards the enemy party and by its optional explosion that the player can trigger.

How it works is a `Prefabs/Objects/IceSphere` gets instantiated and after a bunch of animation logic (after DoCommand 2), it ends up being rooted at (-4.5, 1.75 * x + 0.25 * (1.0 - x), 0.0) where x is `barfill` if it's 0.9 or higher otherwise, it's `barfill` * 0.8 clamped from 0.25 to 1.0.

From there, over the course of 81.0 frames (101.0 frames instead if DoCommand 2 set `commandsuccess` to false), the iceball will move to 10.0 in x via a lerp. During this move, this is where DoDamage 1 call can take effect if the iceball's position + 1.0 goes past an enemy party member's position (they still need to respect some other conditions as the DoDamage table above shows).

However, during the move, the player can at any point interrupt it by pressing the Confirm input when past 34.0 frames. This is only available if DoCommand 2 set `commandsuccess` to true. This is optional: the player may decide to not press the input and let the iceball move all the way to 10.0 in x.

If it is pressed, the iceball immediately stops, `Bee`'s position is set to the iceball's position and DoDamageWithinPos 1 call happens. Since `Bee`'s position is the same as the iceball's for that moment, the iceball's position is what is going to be used as the position to compare the radiuses and determine which enemy party members is affected.

## Logic sequence

- `Bee` MoveTowards (-4.5, 0.0, 0.0) at 2.0x speed
- `Beetle` MoveTowards (-8.2, 0.0, 0.0) at 2.0x speed
- `Moth` MoveTowards (-6.45, 0.0, -0.85) at 2.0x speed
- Camera is set to look at at the left side of the stage
- Yield until `Bee`, `Beetle` and `Moth`'s `forcemove` are done
- `Bee`, `Beetle` and `Moth` have [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them with `overrideflip`, `overrideanim`, `overidefly` and `flip` set to true
- `Moth` animstate set to 105
- `Bee` animstate set to 11 (`Hurt`)
- `Prefabs/Objects/IceSphere` instantiated rooted with a scale of Vector3.zero and stored in a local named iceball with its first child named bsrpite which is rooted and moved offscreen
- `IceRadius` particles played at `Bee`'s position + Vector3.up infinetly with 7.0x uniform scale chiled to `Bee`'s `sprite`
- iceball gets childed to `Bee`'s `sprite`
- `OverworldIce` sound plays
- `mothicenormal` particles plays without sound at `Bee`'s position + 1.0 in y for 3.0 alivetime and 3500 rendersort
- iceball local position set to Vector3.up
- `Bee`'s `bobrange` set to 1.0
- DoCommand 1 call happens
- `Spin` sound plays on `sounds[9]` with 0.5 pitch on loop
- Yield until `doingaction` goes to false. Before each frame yield:
    - A value is calculated being the lerp from the previous value of `barfill` clamped from 0.25 to 1.0 (starting at 0.0) to its current value clamped from 0.25 to 1.0 with a factor of 1/10th of the game's frametime. This controls progression through `barfill`
    - `Bee`'s y `spin` set to 20.0 * the `barfill` progression value
    - iceball scale set to a lerp from Vector3.zero to Vector3.one * 200.0 with a factor of the `barfill` progression value clamped from 0.5 to 1.0
    - `Bee`'s `height` set to a lerp from the existing one to 2.5 with a factor of 0.0075 * the game's frametime
    - `IceRadius`'s ParticleSystem's emmision's rateOverTime set to a lerp from 1.0 to 20.0 with a factor of the `barfill` progression value
    - `sounds[9]`'s pitch set to a lerp from 0.5 to 0.75 with a factor of `barfill`
    - instance.`camoffset` set to a lerp from `defaultcamoffset` to (0.0, 2.5, -11.0) with a factor of the `barfill` progression value
- `sounds[9]` is stopped
- The corrected `barfill` amount is calculated by taking its value (or the value * 0.8 if it's less than 0.9) and clamping it from 0.25 to 1.0
- iceball's scale set to uniform 200.0x * the corrected `barfill` value clamped from 0.5 to 1.0
- instance.`camoffset` set to (0.0, 3.2, -9.85)
- `IceRadius` moved offscreen and destroyed in 5.0 seconds
- iceball gets rooted
- bsprite gets childed to the iceball with a scale of Vector3.zero
- `Bee` is moved offscreen
- `Freeze` sound plays
- `mothicenormal` particles plays without sound at bsprite's position + Vector3.one for 3.0 alivetime and 3500 rendersort with a scale of uniform 2.0x * the corrected `barfill` value
- Yield for 0.5 seconds
- `Beetle` has [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them
- `Beetle`'s `overrideflip` set to true
- `Beetle` MoveTowards (-9.5, 0.0, 0.0) with a stopstate of 116
- `Charge7` sound plays at 0.7 pitch
- Over the course of max 81.0 frames (or until the DoCommand 2 call is done with `doingaction` going false after it started), the iceball moves to its position + 2.0 in y via a BeizierCurve3 with a ymax of 8.0. Before each frame yield, the following happens:
    - If `Beetle`'s `forcemove` is done, it gets ShakeSprite called with 0.1 intensity and 1.0 frametimer
    - `Beetle` animstate set to 116
    - Yield for an additional extra frame
    - If DoCommand 2 call didn't happened yet and we are past 32.0 frames, DoCommand 2 call happens
- `Beetle`'s `sprite` local position is reset to Vector3.zero
- `Beetle` has [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them
- `Moth` animstate set to 102
- If `commandsuccess` is true, `PingShot` sound plays at 0.9 pitch. Otherwise, `Beetle` animstate set to 105
- Over the course of 16.0 frames (26.0 frames instead if `commandsuccess` is false), iceball moves in y to 1.75f * x + 0.25f * (1f - x) where x is the corrected `barfill` value and `Beetle` moves to the iceball initial position before this move -1.0 in x (0.0 for y and z)
- ShakeScreen with an ammount of 0.2 * the corrected `barfill` value for 0.5 time with dontreset
- A SpinAround gets added to iceball
- Camera reset to default
- `hitpart` (the particles) plays at `Beetle`'s position
- If `commandsuccess` is true:
    - iceball's SpinAround's `itself` set to (-10.0, -3.0, -8.0) * the corrected `barfill` value
    - `BigHit` sound plays
- Otherwise:
    - iceball's SpinAround's `itself` set to (-10.0, -3.0, -8.0) * the corrected `barfill` value / 3.0
    - `Beetle` animstate set to 18 (`KO`)
    - DeathSmoke particles plays at `Beetle`'s position
    - `Damage0` sound plays
- If `commandsuccess` is true:
    - `Beetle` moves to its position + (-1.5, 0.0, 0.0) via an ArcMovement with a height of 3.0 over 40.0 frametime
    - `presskey` set to 4 (Confirm)
    - If the corrected `barfill` value is higher than 0.6, a ButtonSprite is SetUp with the `presskey` input childed to the `GUICamera` with a local positon of (0.0, 2.0, 10.0) and a sortorder of 10. The ButtonSprite is disabled for now
- `spin5` sound plays on `sounds[9]` with 1.5 pitch and 0.9 volume on loop
- Yield for a frame
- Over the course of 81.0 frames (101.0 frames instead if `commandsuccess` is false), the iceball moves to (10.0, its current y position, 0.0). During the move:
    - If `commandsuccess` is true and we are past 24 frames:
        - The ButtonSprite gets enabled
        - The ButtonSprite's `basesprite`'s color is set to grey or white (white if Sin(Time.time * 20.0) is higher than 0.0, grey otherwise)
        - If `presskey` (Confirm) is pressed, the iceball moves breaks out early and the explosion damage will happen
    - DoDamage 2 call happens for each applicable enemy party members if they hadn't received the call yet
- The ButtonSprite is destroyed
- `Beetle` and `Moth` animstate set to 13 (`BattleIdle`)
- `sounds[9]` is stopped
- `Explosion5` sound plays
- `IceShatter` particles plays at the iceball with a scale of uniform 3.0x
- `Bee`'s position is set to the iceball's
- The iceball is destroyed
- If the Confirm input press was accepted earlier:
    - ShakeScreen for 0.3 ammount and 1.25 time with dontreset
    - `mothicenormal` particles played at `Bee`'s position + 1.0 in y with scale of uniform 4.0 * corrected value of `barfill`
    - DamageWithinPos 1 call happens
- Otherwise (`commandsuccess` was false or no Confirm input was pressed), ShakeScreen for 0.2 ammount for 0.75 time with dontreset
- `Bee`'s y `spin` set to 30.0
- `Bee`'s `height` set to 0.0
- Over the course of 61.0 frames, `Bee` moves to their initial position before this action via a BeizierCurve3 with a ymax of 7.0
- `Bee` animstate set to 18 (`KO`)
- `Bee`'s `spin` is zeroed out
- DeathSmoke particles played at `Bee`'s position
- `Death3` sound plays
- Yield for 0.5 seconds
- `Bee`, `Beetle` and `Moth` have [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them, they MoveTowards their initial position before this action with a 2.0x speed with `overrideflip`, `overrideanim`, `overridejump` and `overidefly` set to false
- Yield until `Bee`, `Beetle` and `Moth`'s `forcemove` are done
- `Bee`, `Beetle` and `Moth` have their `flip` set to true and their position set to their initial one before this action
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- Yield for a frame
- [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) is called with 0, 1 and 2 (normally `Bee`, `Beetle` and `Moth`)
