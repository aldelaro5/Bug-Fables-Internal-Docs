# AIAttack
This action coroutine involves the `aiparty` doing its attack which is hardcoded according to its [animid](../../../Enums%20and%20IDs/AnimIDs.md). 

When the coroutine starts, all frames will be yielded as long as any enemy party member is in a [forcemove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove). Once they are all done, the actual logic starts.

## Preparation

- `action` is set to true switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- `combo` is set to 1
- [ReorganizeEnemies(true)](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called which sets `enemydata` to be all the alive enemies ordered by their x position
- The position of the `aiparty` is saved locally as the starting position

## Attack
This section depends on the [animid](../../../Enums%20and%20IDs/AnimIDs.md) of `aiparty`.

### `ShielderAnt`

- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to (-1.77, 0.0, -0.55) with a multiplier of 2.0
- Yield all frames until aiparty.`forcemove` is done
- aiparty animstate set to 105
- aiparty's y `spin` set to 30.0
- `Woosh` sound plays at 1.2 pitch on loop
- Yield for 0.5 seconds
- `Woosh` sound stops
- aiparty.`spin` zeroed out
- `Toss8` sound plays
- `Toss2` sound played and has its AudioSource returned by PlaySound on `sounds[3]` with 0.8 pitch on loop. NOTE: This unexpectedly will return null if MainManager.`soundvolume` is 0.0 (the player muted the sound volume) which will cause an exception to be thrown
- `Prefabs/Objects/Shield` gets instantiated
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) is `Ground` are selected for targetting via [GetTargetList](../../Actors%20states/Targetting/GetTargetList.md)
- aiparty animstate set to 102
- Over the course of 40.0 frames:
    - `Prefabs/Objects/Shield` position is lerped from aiparty position + 1.25 in y to + 15.0 in x
    - On each frame, `Prefabs/Objects/Shield` z angles increases by 10.0 * MainManager.`framestep`
    - The ParticleSystem in the `Prefabs/Objects/Shield` prefab (an underglow copy of the shield) has its MainModule's startRotationMultiplier set to the new shield's z angle
    - On the first frame the shield x position becomes the same or higher than a new target's x position (the shield passes through them in x), [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the target for a damageammount of 1 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped without properties and block. This means each target selected earlier will get damages
    - The `Toss2` AudioSource's volume gets lerped from 1.0 to 0.0 and then multiplied by MainManager.`soundvolume`. NOTE: As explained above, if MainManager.`soundvolume` is 0.0 (muted), this is where an exception will be thrown
- Yield for 0.5 seconds
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) is `Flying` are selected for targetting via [GetTargetList](../../Actors%20states/Targetting/GetTargetList.md), but the order is reversed after returned
- Over the course of 40.0 frames:
    - `Prefabs/Objects/Shield` position is lerped from its position to aiparty position + 1.25 in y
    - On each frame, `Prefabs/Objects/Shield` z angles decreases by 10.0 * MainManager.`framestep`
    - The ParticleSystem in the `Prefabs/Objects/Shield` prefab (an underglow copy of the shield) has its MainModule's startRotationMultiplier set to the new shield's z angle
    - On the first frame the shield x position becomes the same or higher than a new target's x position (the shield passes through them in x), [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the target for a damageammount of 1 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped without properties and block. This means each target selected earlier will get damages
    - The `Toss2` AudioSource's volume gets lerped from 0.0 to 1.0 and then multiplied by MainManager.`soundvolume`. NOTE: As explained above, if MainManager.`soundvolume` is 0.0 (muted), this is where an exception will be thrown
- `sounds[3]` stops playing
- `Ding2` sound plays
- The `Prefabs/Objects/Shield` gets destroyed
- aiparty animstate set to 104
- Yield return a SlowSpinStop call on aiparty with a spinammount of -50.0 in y and a frametime of 100.0
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to its position before this attack with a multiplier of 2.0, a state of 1 (`Walk`) and a stopstate of 0 (`Idle`)
- Yield all frames until aiparty.`forcemove` is done
- aiparty's position is set to the starting position before this attack
- aiparty.`flip` set to true
- aiparty animstate set to 13 (`BattleIdle`)

### `Gen`

All enemy party members whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` are selected for targetting via [GetTargetList](../../Actors%20states/Targetting/GetTargetList.md). If there's at least one potential target:

- `eri`'s entity is obtained (it's the first child of aiparty)
- A new sprite object is created at `eri`'s position + 3.25 in y without a sprite for now
- `eri` animstate set to 4 (`ItemGet`)
- A random [item](../../../Enums%20and%20IDs/Items.md) sprite among 3 is selected to be the sprite's `sprite` with the following weighted odds:
    - 3/8: `PoisonDart`
    - 3/8: `NumbDart`
    - 2/8: `SleepBomb`
- `ItemHold` sound plays
- Yield for 1.0 second
- `eri` animstate set to 0 (`Idle`)
- aiparty animstate set to 100
- `Toss12` sound plays
- A random target is selected among the target list obtained earlier
- Depending on the item sprite set earlier, a move is performed:
    - `PoisonDart`:
        - The item object z angle is set to 135.0
        - Over the course of 30.0 frames, the item object position is lerped from aiparty position + (0.75, 1.0, -0.1) to the target position + their `cursoroffset` + (0.0, -0.25 + their `height`, -0.1)
        - `PoisonEffect` sound plays
        - [DoDamage](../../Damage%20pipeline/DoDamage.md) called without attacker to the target for a damageammount of 2 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a property of [Poison](../../Damage%20pipeline/AttackProperty.md) without block
    - `NumbDart`:
        - The item object z angle is set to 135.0
        - Over the course of 30.0 frames, the item object position is lerped from aiparty position + (0.75, 1.0, -0.1) to the target position + their `cursoroffset` + (0.0, -0.25 + their `height`, -0.1)
        - DeathSmoke particles plays at the item object
        - [DoDamage](../../Damage%20pipeline/DoDamage.md) called without attacker to the target for a damageammount of 2 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a property of [Sleep](../../Damage%20pipeline/AttackProperty.md) without block
    - `SleepBomb`:
        - Over the course of 45.0 frames, the item object position moves from aiparty position + (0.75, 1.0, -0.1) to the target position + their `cursoroffset` + (0.0, -0.25 + their `height`, -0.1) via a BeizierCurve3 with a ymax of 4.0. On each frame, the item object z angle increases by 3.0 * MainManager.`framestep`
        - DeathSmoke particles plays at the target with a size of 4.0x
        - [DoDamage](../../Damage%20pipeline/DoDamage.md) called without attacker to the target for a damageammount of 3 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a property of [Sleep](../../Damage%20pipeline/AttackProperty.md) without block
        - For each enemy party members other than the target whose SqrDistance from the target to their position + their `freezeoffset` + their `heigth` in y falls within 15.5, [DoDamage](../../Damage%20pipeline/DoDamage.md) called without attacker to the enemy party member for a damageammount of 1 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a property of [Sleep](../../Damage%20pipeline/AttackProperty.md) without block
        - `Explosion` sound plays
        - ShakeScreen called with an ammount of 0.2 for 0.5 time
- The item object gets destroyed
- Yield for 0.5 seconds
- aiparty animstate set to 0 (`Idle`)

### `Zasp`

- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` are selected for targetting via [GetTargetList](../../Actors%20states/Targetting/GetTargetList.md)
- If there's at least 1 target available, the following happens 2 times:
    - A random target is selected amount the target list obtained earlier
    - aiparty.`specialanim` is set to a new ZaspWarp coroutine  without appear
    - Yield all frames until aiparty.`specialanim` is done
    - aiparty.`specialanim` is set to a new ZaspWarp coroutine  with appear at the target + (+ or - 1.5 determined with a 50/50 RNG check, 0.5, -0.1)
    - Yield for a frame
    - aiparty [FaceTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) the target
    - Yield all frames until aiparty.`specialanim` is done
    - `Toss3` sound plays
    - aiparty animstate set to 102
    - Yield for 0.15 seconds
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) called without attacker to the target for a damageammount of 2 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped without property or block
    - Yield for 0.33 seconds
- If there was at least one possible target:
    - aiparty.`specialanim` is set to a new ZaspWarp coroutine  without appear
    - Yield all frames until aiparty.`specialanim` is done
    - aiparty.`specialanim` is set to a new ZaspWarp coroutine  with appear at their position before this attack
    - aiparty.`flip` set to true
    - Yield all frames until aiparty.`specialanim` is done
    - aiparty animstate set to 13 (`BattleIdle`)

### `LadybugKnight`

- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` will be set as the target (`enemydata[0]` is set as falback if no grounded enemies exists)
- aiparty.`overrideflip` is set to true
- CameraFocus is called on the targetted enemy's battleentity's position which changes instance.`camoffset` to (0.0, 2.6, -6.0), instance.`camtarget` to null and instance.`camtargetpos` to the position of the targetted enemy
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-1.25 * the targetted enemy's `size`, 0.0, -0.1) with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 100
- Yield all frames while aiparty is in a `forcemove`
- Yield for 0.25 seconds
- Damages is only dealt if the targetted enemy's [position](../../Actors%20states/BattlePosition.md) was indeed `Ground` via a [DoDamage](../../Damage%20pipeline/DoDamage.md) call without attacker to the targetted enemy for a damageammount of 2 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a property of [Pierce](../../Damage%20pipeline/AttackProperty.md) without block. No damage is dealt if the enemy wasn't grounded (meaning none existed in the first place)
- Yield for 20.0 frames
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- Yield for 0.33 seconds
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 0 (`Idle`)
- Yield all frames while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier

### `KungFuMantis`

- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` will be set as the target (`enemydata[0]` is set as falback if no grounded enemies exists)
- aiparty.`overrideflip` is set to true
- CameraFocus is called on the targetted enemy's battleentity's position which changes instance.`camoffset` to (0.0, 2.6, -6.0), instance.`camtarget` to null and instance.`camtargetpos` to the position of the targetted enemy
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-1.25 * the targetted enemy's `size`, 0.0, -0.1) with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 100
- Yield all frames while aiparty is in a `forcemove`
- Yield for 0.25 seconds
- Damages is only dealt if the targetted enemy's [position](../../Actors%20states/BattlePosition.md) was indeed `Ground` via a [DoDamage](../../Damage%20pipeline/DoDamage.md) call without attacker to the targetted enemy for a damageammount of 1 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped without properties and block. No damage is dealt if the enemy wasn't grounded (meaning none existed in the first place)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- Yield for 0.33 seconds
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 5 (`Angry`)
- Yield all frames while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier

### `AntCapitain`

- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` will be set as the target (-1 is set as falback if no enemies exists in either `position`)
- If a target was found (-1 wasn't returned):
    - aiparty.`overrideflip` is set to true
    - aiparty.`overridejump` is set to true
    - MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-2.5, 0.0, -0.15) with a multiplier of 2.5
    - Yield all frames while aiparty is in a `forcemove`
    - aiparty.`overrideanim` is set to true
    - aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 100
    - [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to unlock the `rigid`
    - Yield for 0.33 seconds
    - If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) was `Flying`, [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the aiparty with aiparty.`jumpheight` * 1.2 as the height followed by the `Jump` sound being played
    - Yield for 0.33 seconds
    - The `Woosh3` sound is played
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targgeted enemy with a damageammount of 3 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped without properties and block
    - Yield for 0.5 seconds
    - aiparty.`overrideanim` is set to false
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 13 (`BattleIdle`)
    - Yield all frames while aiparty is in a `forcemove`
    - aiparty's position is set to the starting position saved earlier
    - Yield for a frame
    - aiparty.`flip` is set to true
    - aiparty.`overrideflip` is set to false

### `Madeleine`

- aiparty.`overrideflip` is set to true
- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` will be set as the target (`enemydata[0]` is set as falback if no grounded enemies exists)
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) isn't `Ground` (meaning none was found), the attack won't do any damage and its logic will end after doing the following:
    - aiparty.`overrideanim` is set to false
    - aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 8 (`Happy`)
    - Yield for 0.5 seconds
    - aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- The logic proceed, starting by calling MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-1.5, 0.0, -0.15) with a multiplier of 2.5
- Yield all frames while aiparty is in a `forcemove`
- aiparty.`overrideanim` is set to true
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 100
- The `Woosh3` sound is played at 0.9 pitch
- Yield for 0.5 seconds
- The `Woosh4` sound is played
- The following happens twice in sucession (the `Woosh4` sound is played at 1.1 pitch between the 2 iterations):
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targgeted enemy with a damageammount of 2 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a `Pierce` property without block
    - If the targetted enemy's `weight` is less than 100, [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on its battleentity
    - The targetted enemy's battleentity.`spin` is set to (0.0, 15.0, 0.0)
    - Yield for 0.5 seconds
- All frames are yielded while the targetted enemy's battleentity's y position is above 0.15
- The targetted enemy's battleentity.`spin` is zeroed out
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- aiparty.`overrideanim` is set to false
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 0 (`Idle`)
- Yield all frames while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier

### `Maki`

- aiparty.`overrideflip` is set to true
- aiparty.`overridejump` is set to true
- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` will be set as the target (`enemydata[0]` is set as falback if no enemies exists in either `position`)
- MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-2.5, 0.0, -0.1) with a multiplier of 2.5
- Yield all frames while aiparty is in a `forcemove`
- aiparty.`overrideanim` is set to true
- [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to lock the `rigid`
- The `MakiJump1` sound is played
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 114
- Yield for 0.45 seconds
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 112
- The `MakiJump2` sound is played
- instance.`campspedd` is set to 0.1
- instance.`camoffset` is incremented by (0.0, 1.25, -1.25)
- Over the course of 11 frames (tracked by the game's frametime with a local counter), aiparty's position is lerped from the one it had before the first iteration to the same position incremented by (1.0, 4.0, 0.0) with the factor being the progression of those 11 frames. This essentially makes maki move upward and slightly to the right
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 106
- The `MakiJump3` sound is played
- Over the course of 11 frames (tracked by the game's frametime with a local counter), aiparty's position is lerped from the one it had before the first iteration to the same position incremented by (0.0, -4.0, 0.0) with the factor being the progression of those 11 frames. This essentially makes maki move down to the position he had before the previous 11 frames lerp
- DeathSmoke is called with the targgeted enemy's battleentity's position with a size of (2.0, 2.0, 2.0)
- ShakeScreen is called with an amount of 0.2 for 0.5 frames
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) isn't `Underground` (which can only happen if no `Ground` or `Flying` enemy party member existed and the first one happened to be `Underground`), [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targgeted enemy with a damageammount of 6 + amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped with a `Pierce` property and no block
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 107
- 0.65 seconds are yielded
- aiparty.`overrideanim` is reset to false
- [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to unlock the `rigid`
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved at the start of the coroutine with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 13 (`BattleIdle`)
- All frames are yielded while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved at the start of the coroutine
- A frame is yielded
- aiparty.`flip` is set to true
- aiparty.`overrideflip` is set to false

## Enemy drop
This section only occurs if a non zero amount of damages was dealt to an enemy party member.

For each targets whose [position](../../Actors%20states/BattlePosition.md) was `Flying` while its [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) is false, the following happens:

- `startdrop` is set to true
- If the targetted enemy party member doesn't have the `ToppleAirOnly` in its [weakness](../../Actors%20states/Enemy%20features.md#weakness) or it does, but with a [Topple](../../Actors%20states/BattleCondition/Topple.md) condition, the following happens on the enemy:
    - StopAllCoroutines is called on the battleentity
    - battleentity.`droproutine` is set to a new [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) coroutine starting on the battleentity
    - Yield all frames while battleentity.`droproutine` is in progress
    - The enemy party member's [position](../../Actors%20states/BattlePosition.md) is set to `Ground`
- The enemy party member's entity's `droproutine` is set to null (it already should be anyway)
- `startdrop` is set to false

> NOTE: If `Gen` is the aiparty and they call [DoDamage](../../Damage%20pipeline/DoDamage.md) with a [Sleep](../../Damage%20pipeline/AttackProperty.md) property, it's possible that [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) requests a drop on the target even if they are on `Ground` and their `cantfall` is true. If this happens, the drop won't have anyone set `startdrop` to true which will cause the game to stall indefinetely

## Ending
`aiattacked` is set to true followed by a [CheckDead](CheckDead.md) starting.
