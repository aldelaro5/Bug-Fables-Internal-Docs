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

### `KungFuMantis`

- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` will be set as the target (`enemydata[0]` is set as falback if no grounded enemies exists)
- aiparty.`overrideflip` is set to true
- CameraFocus is called on the targetted enemy's battleentity's position which changes instance.`camoffset` to (0.0, 2.6, -6.0), instance.`camtarget` to null and instance.`camtargetpos` to the position of the targetted enemy
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-1.25 * the targetted enemy's `size`, 0.0, -0.1) with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 100
- All frames are yielded while aiparty is in a `forcemove`
- 0.25 seconds are yielded
- Damages is only dealt if the targetted enemy's [position](../../Actors%20states/BattlePosition.md) was indeed `Ground` via a [DoDamage](../../Damage%20pipeline/DoDamage.md) call without attacker to the targetted enemy for 1 damage without properties and block. No damage is dealt if the enemy wasn't grounded (meaning none existed in the first place)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 5 (`Angry`)
- All frames are yielded while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier

### `AntCapitain`

- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` will be set as the target (`enemydata[0]` is set as falback if no enemies exists in either `position`)
- aiparty.`overrideflip` is set to true
- aiparty.`overridejump` is set to true
- MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-2.5, 0.0, -0.15) with a multiplier of 2.5
- All frames are yielded while aiparty is in a `forcemove`
- aiparty.`overrideanim` is set to true
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 100
- [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to unlock the `rigid`
- 0.33 seconds are yielded
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) was `Flying`, [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the aiparty with aiparty.`jumpheight` * 1.2 as the height followed by the `Jump` sound being played
- 0.33 seconds are yielded
- The `Woosh3` sound is played
- [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targgeted enemy for 3 damage without properties and block
- 0.5 seconds are yielded
- aiparty.`overrideanim` is set to false
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 13 (`BattleIdle`)
- All frames are yielded while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier
- A frame is yielded
- aiparty.`flip` is set to true
- aiparty.`overrideflip` is set to false

### `Madeleine`

- aiparty.`overrideflip` is set to true
- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` will be set as the target (`enemydata[0]` is set as falback if no grounded enemies exists)
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) isn't `Ground` (meaning none was found), the attack won't do any damage and its logic will end after doing the following:
    - aiparty.`overrideanim` is set to false
    - aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 8 (`Happy`)
    - 0.5 seconds are yielded
    - aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- The logic proceed, starting by calling MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-1.5, 0.0, -0.15) with a multiplier of 2.5
- All frames are yielded while aiparty is in a `forcemove`
- aiparty.`overrideanim` is set to true
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 100
- The `Woosh3` sound is played at 0.9 pitch
- 0.5 seconds are yielded
- The `Woosh4` sound is played
- The following happens twice in sucession (the `Woosh4` sound is played at 1.1 pitch between the 2 iterations):
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targgeted enemy for 2 damage with a `NoExceptions` property without block
    - If the targetted enemy's `weight` is less than 100, [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on its battleentity
    - The targetted enemy's battleentity.`spin` is set to (0.0, 15.0, 0.0)
    - 0.5 seconds are yielded
- All frames are yielded while the targetted enemy's battleentity's y position is above 0.15
- The targetted enemy's battleentity.`spin` is zeroed out
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- aiparty.`overrideanim` is set to false
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the starting position saved earlier with a multiplier of 2.0 using [animstate](../../../Entities/EntityControl/Animations/animstate.md) 1 (`Walk`) and stopping at animstate 0 (`Idle`)
- All frames are yielded while aiparty is in a `forcemove`
- aiparty's position is set to the starting position saved earlier

### `Maki`

- aiparty.`overrideflip` is set to true
- aiparty.`overridejump` is set to true
- The first enemy party member whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `Flying` will be set as the target (`enemydata[0]` is set as falback if no enemies exists in either `position`)
- MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the aiparty to the targetted enemy's battleentity's position + (-2.5, 0.0, -0.1) with a multiplier of 2.5
- All frames are yielded while aiparty is in a `forcemove`
- aiparty.`overrideanim` is set to true
- [LockRigid(true)](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to lock the `rigid`
- The `MakiJump1` sound is played
- aiparty.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 114
- 0.45 seconds are yielded
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
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) isn't `Underground` (which can only happen if no `Ground` or `Flying` enemy party member existed and the first one happened to be `Underground`), [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacked to the targgeted enemy for 6 damages with a `NoException` property and no block
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
This section only occurs if a non zero amount of damages was dealt to an enemy party member whose [position](../../Actors%20states/BattlePosition.md) was `Flying` while its [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) is false:

- `startdrop` is set to true
- If the targetted enemy party member doesn't have the `ToppleAirOnly` in its [weakness](../../Actors%20states/Enemy%20features.md#weakness) or it does, but with a [Topple](../../Actors%20states/BattleCondition/Topple.md) condition, the following happens on the enemy:
    - StopAllCoroutines is called on the battleentity
    - battleentity.`droproutine` is set to a new [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) coroutine starting on the battleentity
    - All frames are yielded while battleentity.`droproutine` is in progress
    - The enemy party member's [position](../../Actors%20states/BattlePosition.md) is set to `Ground`
- `startdrop` is set to false

## Ending
`aiattacked` is set to true followed by a [CheckDead](CheckDead.md) starting.
