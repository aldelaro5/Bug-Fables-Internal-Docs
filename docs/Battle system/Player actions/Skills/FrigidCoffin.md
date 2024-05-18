# FrigidCoffin
Many of the logic here is shared by [Moth's basic attack](../Basic%20attacks/Moth.md). As such, it is assumed that the actor is `Moth`'s player party member.

It is also assumed that this action cannot target a `Flying` target.

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|25.0|[RandomPressKeyTimer](../../Action%20commands/RandomPressKeyTimer.md)|{0.0, 4.0}|
|2|DoCommand 1 set `commandsuccess` to true|120.0|[SequentialKeys](../../Action%20commands/SequentialKeys.md)|{3.0}|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `commandsuccess` to true|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` unclamped - 1 + `combo` after DoCommand 2 (from 0 to 3 inclusive)|[Magic](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|1_b|DoCommand 1 set `commandsuccess` to false|`Moth`'s player party member|`availabletargets[target]`|`Moth`'s `atk` unclamped|null|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|

## Freeze infliction logic
Right after the DoDamage 1_a call happens, there is some custom [Freeze](../../Actors%20states/BattleCondition/Freeze.md) infliction logic that falls outside of [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md). Since this DoDamage call doesn't have a `Freeze` [property](../../Damage%20pipeline/AttackProperty.md), it can't inflict it, but the action still can with this custom logic that differs slightly from the one used in the damage pipeline.

The condition will be inflicted if all of the following are true:

- If `combo` is higher than 2 (all inputs were pressed correctly after DoCommand 2)
- `lastdamage` is higher than 0 (DoDamage 1_a dealt more than 0 damage to the target)
- target's `freezeres` is lower than 100 (it's not immune)
- A resistance test passes with target's `freezeres` - 30 (- 35 instead if `Moth` has `StatusBoost` [medal](../../../Enums%20and%20IDs/Medal.md) equipped).

The resistance test is performed by generating a random number between 0 and 99 inclusive and seeing if this number is at least the calculated resistance. The test pass if it is and fails if it's not. Exceptionally, if the target's `freezeres` was already 30 or below, the test always passes.

These conditions differs from the ones in the damage pipeline logic. Here's the differences:

- The resitence test has 2 changes:
    - The target's `freezeres` - 30 is used instead of using the `freezeres` directly and it also allows any `freezeres` of 30 or below to always pass the test
    - The `StatusBoost` [medal](../../../Enums%20and%20IDs/Medal.md) decreases further `freezeres` by 5 instead of 15. This means that if it is equipped, the resistence used is `freezeres` - 35 instead of `freezeres` - 15
- The infliction doesn't require the target to not have the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition, but it should be noted that this can't happen under normal gameplay

If all of the above conditions are met, the infliction occurs by doing the following:

- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on target to inflict [Freeze](../../Actors%20states/BattleCondition/Freeze.md) for 2 turns
- If target's `originalid` + 1's [endata](../../../TextAsset%20Data/Entity%20data.md#animid-data)'s `hasiceanim` is true, target's `inice` set to true
- If target's [actimmobile](../../Actors%20states/Enemy%20features.md#actimmobile) is false, its `cantmove` is set to 1 (needs one actor turn before being able to act again)
- [Freeze](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on the target
- If target's `frozenlastturn` is true, its `freezeres` increments by 25. Otherwise, target's `freezeres` increments by 13 (18 instead if on [HardMode](../../Damage%20pipeline/HardMode.md))

However, this logic differs from the one used in the damage pipeline in the following ways:

- The condition is inflicted for 2 turns instead of 1
- The condition is added immediately instead of being a [delayed condition](../../Actors%20states/Delayed%20condition.md). This only has a visual difference: the `Prefabs/Particles/Snowflakes` that normally appears won't appear
- The target's `inice` being set to true condition does not account for being at the `GiantLairFridgeInside` [map](../../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../../Enums%20and%20IDs/librarystuff/Areas.md). Additionally, it doesn't reset the target's [weakness](../../Actors%20states/Enemy%20features.md#weakness) to only contain `HornExtraDamage` if the conditions are fufilled
- The damage pipeline logic has no requirements regarding `actimmobile` being false to set `cantmove` to 1. This shouldn't affect anything in practice because other parts of the battle system allows to bypass inaction via `actimmobile`
- The `freezeres` increase match with the exception that it lacks an increase by 70 that the damage pipeline logic can do. It would have done so if target's `originalid` + 1's [endata](../../../TextAsset%20Data/Entity%20data.md#animid-data)'s `hasiceanim` is true while being at the `GiantLairFridgeInside` [map](../../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../../Enums%20and%20IDs/librarystuff/Areas.md)
- If the target had a [Topple](../../Actors%20states/BattleCondition/Topple.md) condition, it is not removed unlike the damage pipeline logic
- If the target had a [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition, it is not removed if 0 damage is dealt while it normally would be in the damage pipeline logic. It is removed otherwise by DoDamage as it usually does. Unintuitively, because the damage pipeline logic was incorrect, this ends up workarounding the issue because it allows DoDamage to enforce the correct waking up logic
- The damage pipeline logic normally includes a [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) of the target if its [position](../../Actors%20states/BattlePosition.md) is `Ground` while having a `height` above 0.05. While this action doesn't feature this in the freeze infliction logic, a variation of it is included after meaning it applies regardless if the infliction happened or not. The only meaningful difference (as this action is assumed to not be possible on `Flying` enemy party members) is the `height` requirements for the drop is being above 0.1 instead of 0.05. This shouldn't affect anything under normal gameplay

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
- `Prefabs/Objects/icepillar` appears at targetentity with x rotation of -90.0, a DialogueAnim with x/y targetscale of 0.5
- If `commandsuccess` is true (action command succeeded):
    - `Moth` animstate set to 111
    - `IceMothHit` sound plays
    - `icepillar`'s DialogueAnim targetscale is set to half of the target's `size`
    - target animstate set to 11 (`Hurt`) and not `digging`
    - target's [position](../../Actors%20states/BattlePosition.md) is set to `Ground`
    - 0.5 seconds yield
    - `Moth` animstate set to 105
    - `icepillar` gets childed to the target's `sprite`
    - Target has LockRigid(true) called
    - Over the course of 15.0 frames, the target is lifted up in y to 1.5 * (1 - target's [weight](../../Actors%20states/Enemy%20features.md) / 100.0 clamped from 0.0 to 1.0) and while this happens, the target's y `spin` is continuously incremented every frame by 10.0 * (1 - target's [weight](../../Actors%20states/Enemy%20features.md) / 100.0) * the ratio of the 15.0 progression
    - `Moth` animstate set to 102
    - DoCommand 2 call happens
    - Frame yield
    - Yield until `doingaction` is false
    - `Moth` animstate is set to 116
    - Frame yield
    - DoDamage 1_a call happens
    - The custom freeze infliction logic mentioned above is performed
    - `icepillar` destroyed
    - ShakeScreen called with ammount of (0.1, 0.1, 0.1) and time of 0.1
    - `IceShatter` particles played with `IceBreak` sound at slightly above the target
    - 0.5 seconds yield
    - target `spin` zeroed out
    - Over the course of 10.0 frames, the target is moved back to its original position
    - StopAllCoroutine on the target's `battleentity`
    - `Moth` animstate set to 118
    - 0.5 seconds yield
    - LockRigid(false) called on the target
    - target's `basestate` restored to what it was before
    - If target's `height` is above 0.1 while its [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) is false, target's `droproutine` is set to a new [Drop](../../../Entities/EntityControl/Notable%20methods/Drop.md) call followed by its [position](../../Actors%20states/BattlePosition.md) set to `Ground`
    - target's `onground` set to false
- Otherwise (action command failed):
    - `icepillar`'s DialogueAnim targetscale is set to (0.4, 0.4, 0.5)
    - `Moth` animstate set to 109
    - `IceMothMiss` sound plays
    - DoDamage 1_b happens
    - 1.2 seconds yield
    - A `Prefabs/Particles/IceShatter` appears slighly up from `Moth` with x rotation of -90.0 and destroyed in 1.0 seconds
    - `Moth` animstate set to 0 (`Idle`)
- If `Prefabs/Objects/icepillar` still exists, its DialogueAnim is set to shrink at 0.05 speed
