# EventTrigger
An [event](../../../Enums%20and%20IDs/Events.md) trigger zone ???

## Data Arrays
- `data[0]`: The [event](../../../Enums%20and%20IDs/Events.md) id this will start when entered
- `data[1]`: If not 0 or doesn't exist, the object is destroyed after triggering it
- `data[2]`: If it's not -1, disables updates ???

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
Updates are disabled if `data[2]` is present and it's not -1 (TODO: why ???) or we are in `pause`, `minipause` or `inevent`.

This update immediately and unconditionally (TODO: what ???) starts the [event](../../../Enums%20and%20IDs/Events.md) id `data[0]` 

Finally, this object gets destroyed.

TODO: There's clearly more to this because it doesn't make sense to unconditionally start the event... ???

## OnTriggerStay
This does nothing if any of the following is true:
- The player isn't present
- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true 

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true.

If we aren't in a `minipause`, a WaitForEvent coroutine starts which does the following:
- `hit` is set to true
- We enter a `minipause`
- Yield frames until `switchcooldown` expires
- Yield another additional frame
- Call CancelAction on the player
- Yield a frame
- The [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[0]` is started with this object as the caller
- `hit` is set to false
- If `data[1]` isn't 0 or doesn't exist, the object is destroyed
