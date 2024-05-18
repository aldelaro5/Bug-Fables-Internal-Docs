# `IceDrill`
This is a [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) with `Beetle` and `Moth` and as such, it is assumed that the attacker is either one and it also means that [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) won't consume their actor turns like normal.

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|250.0|[SequentialKeys](../../Action%20commands/SequentialKeys.md)|{6.0}|
|2|DoCommand 1 set `combo` to a value higher than 2|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random integer (cast to float) between 4 and 6 (Confirm, Cancel or Switch input)}|

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `combo` to a value higher than 2, done 3 times|null|`availabletargets[target]`|[GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) for `Beetle` and `Moth` / 2.0 * `barfill` rounded to nearest (to nearest even if ending in .5) + 1|[Pierce](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
|1_b|Either one of the following happened:<ul><li>DoCommand 1 set `combo` to a value of 2 or lower</li><li>DoCommand 2 set `commandsuccess` to false</li></ul>|null|`availabletargets[target]`|1|null|empty array|false|
|2|DoCommand 2 set `commandsuccess` to true|null|`availabletargets[target]`|[GetMultiDamage](../../Damage%20pipeline/GetMultiDamage.md) for `Beetle` and `Moth` / 2.0 * `barfill` rounded to nearest (to nearest even if ending in .5) + 2|null|empty array|false|

## Additional flip logic
DoDamage 1_a doesn't have a `Flip` [property](../../Damage%20pipeline/AttackProperty.md), but the action comes with custom logic that does a reduced version of what the property does and it is also done 3 times, each being done right after DoDamage 1_a call. It only concerns the [conditions](../../Actors%20states/Conditions.md) management and the `basestate` set in case of a [Topple](../../Actors%20states/BattleCondition/Topple.md).

Flip is called on the `target` enemy party member. This method features the same [flip handling](../../Damage%20pipeline/CalculateBaseDamage.md#flip-handing) logic than the damage pipeline, but with 3 key differences:

- It doesn't have any logic about the damage effects (it doesn't apply here)
- It doesn't feature a [DropItem](../../Actors%20states/Enemy%20party%20members/DropItem.md) call if the enemy party member was holding an item
- It doesn't feature setting `isdefending` to false meaning it will not break their guarding

The rest (including the toppling management and the [Flipped](../../Actors%20states/BattleCondition/Flipped.md) condition infliction) are the same.

## Logic seqeunce

- Both `Beetle` and `Moth` have [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them with `flip` and `overrideflip` set to true
- `Beetle` MoveTowards (-2.95, 0.0, -1.45) at 2.0x speed
- `Moth` MoveTowards (-4.25, 0.0, -2.0) at 2.0x speed
- Yield until `Beetle` and `Moth`'s `forcemove` are done
- Both `Beetle` and `Moth` have `flip` set to true and `overrideflip` set to false
- Camera moves slowly to the midpoint between the attacker and the `target` enemy party member
- `Moth` animstate set to 108
- `Prefabs/AnimSpecific/mothbattlesphere` instantiated rooted at `Moth`'s position + (-0.75, 1.55, 0.0) with 90.0 angle in x
- `Prefabs/Objects/SingleSphere` instantiated childed to the `Prefabs/AnimSpecific/mothbattlesphere` with a local position of Vector3.zero and a MeshRenderer's materiral.color of pure white with half transparency
- `Prefabs/AnimSpecific/mothbattlesphere` moves via an ArcMovement to its position + (2.0, -5.0, 0.0) with 3.0 height over the course of 40.0 frametime
- `Prefabs/AnimSpecific/mothbattlesphere` destroyed in 2.0 seconds
- `Dig` sound plays
- `checkingdead` set to a new KabbuDig call on `Beetle`
- Yield until `checkingdead` is done
- Yield for 0.5 seconds
- [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on `Beetle`
- `Beetle` moves towards (MainManager.MoveTowards) the `target` enemy party member + -0.1 in z over 75.0 frametime with smooth
- DoCommand 1 call happens
- Yield for 1.0 seconds
- Yield until `doingaction` is false
- `commandsuccess` is set to whether or not `combo` is higher than 2 (the command is considered succeeded for this specific action if it is)
- `DigPop` sound plays
- `DirtExplode` particles plays at the `target` enemy party member position
- ShakeScreen for 0.1 ammount for 0.75 time with dontreset
- If `commandsuccess` is true:
    - `WaspDrill` sound plays on loop
    - `Prefabs/Objects/IceDrill` instantiated rooted at the `target` enemy party member + -5.0 in y with -90.0 angle in x
    - `Prefabs/Objects/IceDrill` moves towards (MainManager.MoveTowards) the `target` enemy party member over the course of 10.0 frametime with local (this will smoothly reveal the object from the ground)
    - `Moth` animstate set to 101
    - `Prefabs/Objects/IceDrill` z scale set to 0.75
    - Camera moves to look at the `target` enemy party member + 2.0 in y
    - `Moth` animstate set to 108
    - `Prefabs/Objects/IceDrill` gets a SpinAround added with an `itself` of (0.0, 0.0, -1.5 * `combo`)
    - `Beetle` is set to not be `digging`
    - `Beetle` y position decreases by 3.0
    - If the `target` enemy party member's [weight](../../Actors%20states/Enemy%20features.md#weight) is less than 100.0:
        - [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them
        - Their y `spin` is set to 2.5 * `combo`
    - `target` enemy party member gets `overrideanim` and `overridejump` set to true
    - `target` enemy party member animstate set to 11 (`Hurt`)
    - The following is done 3 times:
        - If the `target` enemy party member's [weight](../../Actors%20states/Enemy%20features.md#weight) is less than 100.0, they will move to 1.5 in y via an ArcMovement with 4.0 height over 40.0 frametime with their `onground` set to false
        - `ShieldHit` sound plays with 0.65 volume
        - DoDamage 1_a call happens
        - Flip is called on the `target` enemy party member (see the section above for more details)
        - If this is the last (third) iteration, DoCommand 2 call happens
        - `Moth` animstate set to 104
        - Yield for 0.5 seconds
        - Yield for 0.1 seconds
    - Yield until `doingaction` is false
    - `Beetle`'s `sprite` local position set to Vector3.zero
    - `target` enemy party member's `spin` zeroed out
    - `WaspDrill` sound stopped
    - If `commandsuccess` is true (set by DoCommand 2):
        - `BigHit` sound plays
        - `OverworldIce` sound plays at 0.9 pitch
        - `DirtExplode` particles plays at the `target` enemy party member
        - `mothicenormal` particles plays at `Prefabs/Objects/IceDrill` + 2.0 in y with a uniform scale of 2.5x
        - `IceShatter` particles plays at `Prefabs/Objects/IceDrill` + 2.0 in y with a uniform scale of 2.5x
        - `Prefabs/Objects/IceDrill` destroyed
        - ShakeScreen with an ammount of 0.2 for 0.85 time with dontreset
        - `Beetle`'s y `spin` set to 30.0
        - `Beetle`'s `overrideflip` set to false
        - `Beetle` animstate set to 119
        - If the `target` enemy party member's [weight](../../Actors%20states/Enemy%20features.md#weight) is less than 100.0, they will move to 0.0 in y via an ArcMovement with 7.5 height over 60.0 frametime with their `onground` set to false
        - DoDamage 2 call happens
        - `Beetle` moves to its initial position before this action + Vector3.up via an ArcMovement with a height of 10.0 over 60.0 frametime
        - Yield for 1.0 seconds
- FlipAngle(true) called on `Beetle`
- Camera reset to default slowly
- `Beetle`'s `sprite` local position set to Vector3.zero
- If `commmandsuccess` is false (meaning DoCommand 1 set `combo` to 2 or lower OR DoCommand 2 set `commandsuccess` to false):
    - `Moth` animstate set to 109 if DoCommand 1 didn't set `combo` to higher than 2
    - `DirtExplode` particles plays at the `target` enemy party member
    - If `Prefabs/Objects/IceDrill` exists:
        - `IceShatter` particles plays where it is + 1.0 in y
        - It is destroyed
    - DoDamage 1_b call happens
    - `Beetle` moves to its initial position before this action via an ArcMovement with a height of 10.0 over 60.0 frametime
    - `Beetle` animstate set to 11 (`Hurt`)
    - Yield for 1.0 seconds
- Both `Beetle` and `Moth` have the following happen to them:
    - MoveTowards their initial position before this action at 2.0x speed with a stopstate of 13 (`BattleIdle`)
    - `overrideflip` set to true
    - `overrideanim` set to false
    - `spin` zeroed out
    - [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on them
- Yield until `Beetle` and `Moth`'s `forcemove` are done
- Both `Beetle` and `Moth` have the following happen to them:
    - `overrideanim` set to false
    - `flip` set to true
    - position set to initial position before this action
- [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on `target` enemy party member
- `target` enemy party member's `overrideanim` set to false
- [MultiSkillMove](../../Actors%20states/MultiSkillMove.md) is called with 1 and 2 (normally `Beetle` and `Moth`)
