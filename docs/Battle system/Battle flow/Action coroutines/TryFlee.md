# TryFlee
This action coroutine allows the player to attempt to flee from the battle if an action command is passed. This allows to go directly to the [ReturnToOverworld](../Terminal%20coroutines/ReturnToOverworld.md) in a [terminal flow](../Update%20flows/Terminal%20flow.md#terminal-flow) if the action command is passed. If the action command is failed, all player party members gets all their action turns exhausted which leads to [EndPlayerTurn](../EndPlayerTurn.md) earlier than usual.

## Command preparation

- `action` is set to true changing to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- `commandsuccess` is reset to false
- instance.`showmoney` is set to 100000000.0 which effectively makes the berry count HUD element shown permanently for now
- The instance.`camtargetpos` is set to (-4.5, 0.0, 2.5) with an instance.`camspeed` of 0.075
- 10 RigidBodies are initialised to hold lost berries (they each get a SpriteRenderer using the `MoneySmall`'s `itemsprites`) childed to the `battlemap` in kinematic mode, but initially placed offscreen at (0.0, 999.0, 0.0)
- The `FlipNoise` sound is played
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), the following adjustements happens on their battleentity (this setups their walk in place animations):
    - `anim`.speed is set to 2.0
    - `overridejump` is set to true
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called
    - `flip` is set to false
    - `overrideanim` is set to true
    - [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 1 (`Walk`)
- 0.4 seconds are yielded
- A [DoCommand](../../DoCommand.md) coroutine is started with a commandtype of `SequentialKeys` using 250.0 as the timer and the data being one element whose value is 6 unless the `SpeedUp` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped which makes the value 3 instead

## Performing the action command
This section is in effect as long as `doingaction` is true. It's where the berry loss happens periodically when applicable. A frame is yielded at the end of every cycle when this section is in effect.

In order for a berry to be lost, the following must be true:

- instance.`money` must be above 0
- The `SecurePouch` [medal](../../../Enums%20and%20IDs/Medal.md) is unequipped
- [flags](../../../Flags%20arrays/flags.md) 162 is false (We aren't in a B.O.S.S or Cave of Trials session)
- A certain amount of frames has passed (tracked by a local framecounter using instance.`framestep` as the increment)

The amount of frames that needs to pass to loose a berry depends on `estimatedexp`. It is calculated by taking (30 - `estimatedexp`) / 2, clamping it from 5 to instance.`neededexp` / 2 and finally, multiply the result by 2.0 which also casts it to a float for the counter comparison. What this intuitively mean is that the amount of frames before a berry is lost is always between 5 and 15, but the base amount decreases the more EXP has been accumulated so far. As for the upper clamping bound, it doesn't do anything because the minimum value of instance.`neededexp` is normally 100 making its half 50 which is higher than 15, the upper bound of the base value if `estimatedexp` is 0.

All in all, this can be reduced to just (30 - `estimatedexp`) / 2 clamped from 5 making the overall result from 5 to 15 frames between loosing a berry.

If no berry losses occurs, the local counter is incremented by instance.`framestep`.

If a berry loss occurs, the following is done on one of the 10 RigidBodies initialised earlier (the index starts at 0 and increments each berry loss, resetting to 0 when reaching 10):

- position is set to a randomly selected player party member's battleentity's position
- kinematic mode is turned off
- velocity is set to RandomItemBounce(4.0, 15.0)

The berry loss ends by decrementing instance.`money` and resetting the local frame counter to 0.0.

## Handling the action command result
The rest of the coroutine depends on `commandsuccess` being true or not indicating the command was succeeded or failed. In either outcome, all RigidBodies initialised earlier are destroyed at the very end.

### Success
This section is applicable if `commandsuccess` is true:

- MainManager.`battleresult` is set to false
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on their battleentity
- 0.45 seconds are yielded
- The `Flee` sound is played
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), the following happens on their battleentitty:
    - The y position is set to 0.0
    - The `rigid`'s gravity is disabled
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called to move to (-20.0, 0.0, 0.0) with a multiplier of 5.0
- All frames are yielded until the `forcemove` of the battleentity of the last player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md) goes to false (meaning the last player party member that was moved is done with its `forcemove`)
- [flagvar](../../../Flags%20arrays/flagvar.md) 42 (number of fleed battles) is incremented
- [ReturnToOverworld](../Terminal%20coroutines/ReturnToOverworld.md) is called with flee

### Fail
This section is applicable if `commandsuccess` is false:

- The `Fail` sound is played
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), the following happens on their battleentitty (this undo what was done on the command preparations with some more animations):
    - `anim`.speed is reset to 1.0
    - `overrideanimspeed` is set to false
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called
    - [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`)
- 0.1 seconds are yielded
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), the following happens on their battleentitty:
    - DeathSmoke is called with the battleentity
    - [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 18 (`KO`)
    - The `Drop` sound is played on the entity (not the battleentity)
- If [flags](../../../Flags%20arrays/flags.md) 162 is false (we aren't in a B.O.S.S or Cave of Trials session), all of the 10 RigidBodies will count as a berry loss meaning up to 10 berries are lost (or instance.`money` berries is lost if less than 10 remained). The following happens for each loss on the concerned RigidBody:
    - position is set to a randomly selected player party member's battleentity's position
    - kinematic mode is turned off
    - velocity is set to RandomItemBounce(4.0, 15.0)
    - instance.`money` is decremented
    - A frame is yielded
- 0.85 seconds are yielded
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- For each player party member that isn't [IsStopped](../../Actors%20states/IsStopped.md), the following happens on their battleentitty:
    - `overridejump` is set to false
    - `overrideanim` is set to false
    - `flip` is set to true
    - The player party member's `cantmove` is set to 1 except for the `currentturn` one where it's set to 0 (this is because its turn will be exhausted soon so all member's `cantmove` will end up at 1)
- A frame is yielded
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- instance.`showmoney` is set to -1 which hides the berry count HUD
- [EndPlayerTurn](../EndPlayerTurn.md) is called (which will increment the `currentturn` player party member so all members's `cantmove` ends up at 1)
- `action` is set to false switching back to a [controlled flow](../Update%20flows/Controlled%20flow.md)
