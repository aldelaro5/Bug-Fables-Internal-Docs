TODO: simplify the collider tags effects and describe more precisely what colliding with the beemerang does

# DoActionTap
This is a coroutine that where the leader party member's tap action will be performed. It's called a tap action because it involves holding the ability inputs for less than 20.0 frames in [GetInput](../GetInput.md) which is considered a tap.

Some actions may go differently when a second tap is detected shortly after the first. This is part of why this is a coroutine: it allows to wait and check for a second tap.

At any point however, the action may be cancelled by calling [CancelAction](CancelAction.md) which will among other things, forcefully stop this coroutine (the coroutine is meant to be stored in `actioncoroutine` which is set to null when completed).

It's also possible this coroutine is called from [DoActionHold](DoActionHold.md), but only if the conditions for performing the hold actions aren't met. In that case, it's treated the same as if it was a tap action.

It should also be noted that it's not guaranteed the action ends up happening because any requires the player to not be in a `submarine` and each action may have their own set of requirements that must be fufilled (the same goes for their double tap variant).

This page describes what the coroutine ends up doing to perform each action.

## Setup
The coroutine starts by resetting `lastactionid` to 0 (this will only be used later when `Beetle` performs an action since they get less `actioncooldown`).

From there, it's possible the action isn't performed if the player is in a `submarine`. If this happens, `digging` is toggled. TODO: why ??? If they aren't in a `submarine`, the action is performed.

No matter what though, `actioncoroutine` is always set to null at the end.

## Action setup
Here's what happens before any actions:

- entity.`flip` is set to `trueflip`
- entity [StopMoving](../../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) with a targetstate of 0 (`Idle`)
- `actionhold` is reset to 0.0
- entity.`backsprite` is reset to false
- `action` is set to true to mark the tap action being in progress
- `lockkeys` is set to true which locks all of regular movement inputs processing in Update and [GetInput](../GetInput.md) in general
- An angle is obtained via GetAngle that will be used during the action. Here's what the method does:
    - TODO: complex angle math, need to recheck

### `Bee`'s tap action (Beemerang Toss)
This action only happens when `playerdata[0]`'s `animid` (not its entity.`animid`) is 0 which should mean that their entity.[animid](../../Enums%20and%20IDs/AnimIDs.md) is `Bee`.

It also requires that any of the following conditions is fufilled, the action doesn't happen and the coroutine goes directly to the action cleanup section below:

- [flag](../../Flags%20arrays/flags.md) 41 is false (Chapter 1 hasn't ended yet)
- [flag](../../Flags%20arrays/flags.md) 11 is true (obtained Beemerang Toss)

If any of the conditions above are fufilled, the action is performed:

- entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) set to 100
- `Toss` sound plays on entity
- entity's y `spin` set to -30.0 if entity.`flip` is false, 30.0 instead if it's true
- Yield for 0.2 seconds
- entity.`animstate` set to 101
- entity.`spin` zeroed out
- `beemerang` is set to a new `Prefabs/Objects/Beerang` GameObject childed to the player on layer 0 () with an `insideid` set to the current [inside](../../MapControl/Insides.md) (instance.`insideid`) and whose EntityControl has the following properties:
    - `speed`: 0.075
    - `spin`: (0.0, 0.0, 20.0)
    - `flip`: player's entity.`flip`
- entity.`overrideflip` is set to true
- entity.`sprite` y angle is set to the angle obtained earlier in action setup
- `beemerang`.`vectordata[0]` is set to its position + `lastdelta` * 7.5 which is its attempted destination (check the [Beemerang](../../Entities/NPCControl/ObjectTypes/Beemerang.md) documentation to learn more)
- Yield for 0.25 seconds

There is technically no double tap action because the second optional part of the action is the Halt feature which is handled separately in the [Beemerang](../../Entities/NPCControl/ObjectTypes/Beemerang.md) logic. It is where the requirements for it are checked as well as the inputs needed.

### `Beetle`'s tap action (Horn Slash / Horn Dash)
This action only happens when `playerdata[0]`'s `animid` (not its entity.`animid`) is 0 which should mean that their entity.[animid](../../Enums%20and%20IDs/AnimIDs.md) is `Beetle`:

No matter what happens, `lastactionid` is set to 1 which implies `actioncooldown` will have a lesser value later, but the action still requires that the player wasn't `dashing` as otherwise, this happens instead of the action:

- entity [StopMoving](../../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) with a targetstate of -1 (defaults to the `basestate` if their `animstate` was their `walkstate`)
- `dashing` is set to false which stops [DashBehavior](../DashBehavior.md) from being called
- entity.`jumpcooldown` is set to 5.0

However, it should be noted that it's impossible we ever get into this situation. Not only the player `dashing` prevents [GetInput](../GetInput.md) from ever calling this coroutine, if the player was `dashing`, [DashBehavior](../DashBehavior.md) would have stopped the dash already because it would have seen the input was pressed and call StopDash immediately. The logic above is actually incorrect because StopDash more exhaustively clean up things like `tbox` and `smoke` while the logic above doesn't. Effectively, this logic should never happen and if it did, it could leave the dash in a bad state.

If the player wasn't `dashing`, then the action happens:

- entity's `overridejump`, `overrideanim` and `overrideflip` are set to true
- entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) set to 100
- entity.`sprite` y angle is set to the angle obtained earlier in action setup
- `Cut` sound plays at 0.5 volume
- Over the course of the next 15.0 frames, the ability input (without hold) is monitored to see if a second tap is detected. If it is and [flag](../../Flags%20arrays/flags.md) 699 is true (obtained the dash), the frame yields stops and the dash will be done instead of the slash. NOTE: Even if flag 699 is false, the 15.0 frames yields still happens despite the fact that the double tap cannot be registered so no matter what, the Horn Slash always ends up being delayed by 15.0 frames
- `tbox` is initialised to be the BoxCollider of a new rooted GameObject named `cut` with a `BeetleHorn` tag (see the section below for details) and its angles set to the entity's angles. Here are the `tbox` properties:
    - size: (1.0, 1.5, 2.25)
    - center: (0.0, 1.0, 0.0)
    - isTrigger: true

From there, what happens in the action depends on if a double tap was registered earlier or not.

#### Single tap (Horn Slash)
This section only happens if no double tap was registered earlier:

- `tbox` gets destroyed in 0.15 seconds
- Yield for 0.15 seconds

#### Double tap (Horn Dash)
This section happens only if a double tap was registered earlier:

- `spin4` sound plays
- `smoke` is initialised to be the ParticleSystem of a new `Prefabs/Particles/WalkDust` GameObject childed to this player with a local position of (0.0, 0.25, 0.1), an x angle of -90.0 and a Renderer's material.renderQueue of 3001 TODO: ???
- `diggingpart` set to `smoke`
- `smoke` has its ParticleSystem gets some adjustements:
    - MainModule's startlifetime: 1.0 constant curve
    - MainModule's startSize: 0.75 constant curve
    - EmissionModule's rateOverTime: 5.0 constant curve
- `tbox` gets childed to the player entity with a local position of Vector3.zero and the following adjustments:
    - center set to (0.0, 1.5, 1.0)
    - size set to (1.0, 1.5, 1.0)
- entity.`overridejump` set to true
- entity.`animstate` set to 116
- entity gets [Jump](../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 5.0
- Yield for 0.1 seconds
- Yield until entity.`onground` is true, but for up to 120.0 frames where the yields will stop regardless (it's a failsafe)
- entity.`offgroundframes` is reset to 0.0 (reset the coyote time frame counter)
- `dashing` is set to true which means the regular inputs processing Update does will get overriden by [DashBehavior](../DashBehavior.md) from now on
- `dashdelta` is initialised to `lasdelta`.normalized * 4.0 (this doesn't do anything because `dashdelta` will soon get overriden by DashBehavior, but it gives a sensible initial value)
- `dashtarget` is set to `dashdelta` (this is needed for DashBehavior to start with something consistent during movement)
- `smoke` has its ParticleSystem gets some adjustements:
    - MainModule's startSize: 1.1 constant curve
    - EmissionModule's rateOverTime: 10.0 constant curve
- The tag of `tbox` is determined depending on [flag](../../Flags%20arrays/flags.md) 39 (got the upgraded dash):
    - true: `BeetleDash` (see the section below for details)
    - false: `BeetleHorn` (see the section below for details)

TODO: simplify this with a table

#### Effects of `BeetleHorn`
Here are the effects of `BeetleHorn` tag for both single or double tap actions:

- [Hornable](../../Entities/NPCControl/ObjectTypes/Dropplet.md#hornable)'s OnTriggerEnter: allows to move the object
- MusicSpinner's OnTriggerEnter: allows to hit and roll the spinner
- [BreakableRock](../../Entities/NPCControl/ObjectTypes/BreakableRock.md)'s OnTriggerEnter allows to shake, but not destroy the rock
- [SavePoint](../../Entities/NPCControl/ObjectTypes/SavePoint.md)'s OnTriggerEnter: allows to interact with the save to save the game
- [PushRock](../../Entities/NPCControl/ObjectTypes/PushRock.md)'s OnTriggerEnter: allows to move the object
- [CoiledObject](../../Entities/NPCControl/ObjectTypes/CoiledObject.md)'s OnTriggerEnter: allows to uncoil the object trapped
- [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../../Entities/NPCControl/ObjectTypes/WaterSwitch.md)'s OnTriggerEnter: allows to `hit` the switch
- [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md)'s OnTriggerEnter: allows to cut the grass
- [ScrewSwitch](../../Entities/NPCControl/ObjectTypes/ScrewSwitch.md): allows to very slightly actuate the switch, but not enough for useful actuation
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md): allows to break a frozen geizer
- An [Enemy](../../Entities/NPCControl/Enemy.md): allows to move the enemy into a dizzy state
- Any NPCControl whose entity's name has the `ITAH` [modifier](../../Entities/EntityControl/Modifiers.md): allows for the NPCControl to call [Interact](../../Entities/NPCControl/Notable%20methods/Interact.md)
- [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md)'s OnTriggerEnter: allows to make the spring bounce
- Any ShakeHorn: allows to shake the object

#### Effects of `BeetleDash`
Here are the effects of `BeetleDash` tag for the double tap action:

- [BreakableRock](../../Entities/NPCControl/ObjectTypes/BreakableRock.md)'s OnTriggerEnter allows to break the rock
- [SavePoint](../../Entities/NPCControl/ObjectTypes/SavePoint.md)'s OnTriggerEnter: allows to interact with the save to save the game
- [CoiledObject](../../Entities/NPCControl/ObjectTypes/CoiledObject.md)'s OnTriggerEnter: allows to uncoil the object trapped
- [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../../Entities/NPCControl/ObjectTypes/WaterSwitch.md)'s OnTriggerEnter: allows to `hit` the switch
- [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md)'s OnTriggerEnter: allows to cut the grass
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md): allows to break a frozen geizer
- An [Enemy](../../Entities/NPCControl/Enemy.md): allows to move the enemy into a dizzy state and to break them out of freeze
- Any NPCControl whose entity's name has the `ITHD` [modifier](../../Entities/EntityControl/Modifiers.md): allows for the NPCControl to call [Interact](../../Entities/NPCControl/Notable%20methods/Interact.md)
- [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md)'s OnTriggerEnter: allows to make the spring bounce
- Any ShakeHorn: allows to shake the object


### `Moth`'s tap action (Freeze / Icicle)
This action only happens when `playerdata[0]`'s `animid` (not its entity.`animid`) is 0 which should mean that their entity.[animid](../../Enums%20and%20IDs/AnimIDs.md) is `Moth`:

- entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) set to 111
- entity.`overrideflip` set to true
- entity.`sprite` y angle is set to the angle obtained earlier in action setup
- A new `Prefabs/Particles/mothicenormal` GameObject is created rooted at the player position + Vector3.up + `lastdelta` * 2.0 with a y angle of entity.`detect`'s y angle + 90.0, a tag of `Icecle` and a BoxCollider with the following properties:
    - size: (2.0, 1.0, 1.0)
    - center: (0.5, 0.0, 0.0)
    - isTrigger: true
- `OverworldIce` sound plays with 0.8 volume
- The BoxCollider of the `Prefabs/Particles/mothicenormal` GameObject gets destroyed in 0.25 seconds
- `Prefabs/Particles/mothicenormal` GameObject gets destroyed in 1.0 seconds

The `Icecle` tag allows the following to happen:

- [SavePoint](../../Entities/NPCControl/ObjectTypes/SavePoint.md)'s OnTriggerEnter: allows to interact with the save to save the game
- [CoiledObject](../../Entities/NPCControl/ObjectTypes/CoiledObject.md)'s OnTriggerEnter: allows to uncoil the object trapped
- [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../../Entities/NPCControl/ObjectTypes/WaterSwitch.md)'s OnTriggerEnter: allows to `hit` the switch
- [Dropplet](../../Entities/NPCControl/ObjectTypes/Dropplet.md)'s OnTriggerEnter: allows to `hit` the dropplet which turns it into an ice cube
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)'s OnTriggerEnter: allows to freeze the geizer
- An [Enemy](../../Entities/NPCControl/Enemy.md)'s OnTriggerEnter: allows to freeze the enemy or to renew an existing freeze
- [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md)'s OnTriggerEnter: allows to make the spring bounce

The double tap part of the action may happen here if [flag](../../Flags%20arrays/flags.md) 171 is true (obtained Icicle), but if it's false, the action ends here with a yield for 0.1 seconds.

If the flag is true, the double tap is checked here for the next 25.0 frames. It is detected if the ability input is pressed without hold during those 25.0 frames which will cause this wait to stop immedietaly (the wait still happens for 25.0 frames even if the input hasn't been pressed).

If the double tap isn't detected, the action ends immediately.

#### Double tap part (Icicle)
This section only happens if the double tap was registered earlier:

- `Freeze` sounf plays at 0.5 volume and 1.25 pitch
- entity.`overrideanim` set to true
- entity.`animstate` set to 108
- If `icecle` is null:
    - `icecle` is initialised to be a new instance of `Prefabs/Objects/icecle`. This has a tag of `icecle` which has the following effect:
        - Hazard's OnTriggerEnter when `type` is `Water`: Creates an ice platform
        - [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../../Entities/NPCControl/ObjectTypes/WaterSwitch.md)'s OnTriggerEnter: allows to `hit` the switch
        - [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)'s OnTriggerEnter: allows to freeze the geizer
    - `iceclesize` is reset to 0.0
- `icecle` position is set to the player position + (0.0, 4.0, 0.0) + `lastdelta` * 2.0
- Yield all frames until `iceclesize` reaches 1.0 where it is increased by TieFramerate(0.05) before each frame yield (so this lasts ~20.0 frames). Before a frame yield, the following also happens:
    - `icecle` y angle increased by 20.0
    - `icecle` scale set to Vector3.one * `iceclesize`
- DropIcecle called which does the following:
    - `icecle` scale set to Vector3.one
    - `icecle` gets some components added:
        - Rigidbody with constraints set to `FreezeRotation` (this will make the object fall due to gravity)
        - DestroyOnLayer SetUp with `IceShatter` deathparticles for 2.0 particletime on particlelayer 8 (`Ground`) with a partoffset of 0.5 in y and a partangle of -90.0 in x
        - DestroyOnLayer SetUp with `IceShatter` deathparticles for 2.0 particletime on particlelayer 13 (`NoDigGround`) with a partoffset of 0.5 in y and a partangle of -90.0 in x
    - `icecle`'s BoxCollider gets disabled then a new one gets added with the following properties:
        - size: (1.0, 1.5, 1.0)
        - isTrigger: true
    - All [entityonly](../../MapControl/EntityOnly.md) colliders of the current `map` gets their collisions ignored with `icecle`'s new BoxCollider
    - `icecle` gets destroyed in 1.0 seconds
    - `icecle` is set to null
- Yield for 0.2 seconds

## Action cleanup
Here's what happens at the end of every action:

- Yield for a frame
- `lastdelta` is reset to a value that depends on entity.`flip`:
    - true: `CamDir`'s right vector normalized
    - true: `CamDir`'s -right vector normalized (so the left vector normalized)
- `action` is reset to false
- `lockkeys` is reset to false so inputs processing can happen again
- entity's `overrideanim`, `overrideflip` and `overridejump` are reset to false
- ActionCooldown is called which sets `actioncooldown` to a value depending on `lastactionid`:
    - 1 (`Beetle`): 5.0 frames
    - Any other values (`Bee` or `Moth`): 17.0 frames
