# EventTrigger
An [event](../../../Enums%20and%20IDs/Events.md) trigger. It can either unconditionally trigger as a one shot event on the first Update cycle or trigger via OnTriggerStay that starts the event when the player collides with it with the option to destroy the NPCControl after.

## Data Arrays
- `data[0]`: The [event](../../../Enums%20and%20IDs/Events.md) id this will start when triggered
- `data[1]`: If not 0 or doesn't exist, the object is destroyed after triggering it, only applicable if `data[2]` doesn't exist or isn't 1
- `data[2]`: If 1, the event is unconditionally started on the first Update cycle before destroying this NPCControl making it a one shot event. This is optional, if it doesn't exist, this feature is disabled.

## Additional data
- `regionalflag`: If it's positive, it's the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot that is set to true when the player collides with this object.
- `activationflag`: If it's above 0, the [flag](../../../Flags%20arrays/flags.md) slot that is set to true when the player collides with this object. It is not possible to specify 0 and will be treated the same as not providing a value.

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
Updates are disabled if `data[2]` doesn't exist or isn't -1 or we are in `pause`, `minipause` or `inevent`. All Update does is manage the unconditional one shot trigger feature.

If the above is fufilled, the [event](../../../Enums%20and%20IDs/Events.md) id `data[0]` is started with no caller immediately.

Finally, this object gets destroyed.

## OnTriggerStay
This does nothing if any of the following is true:
- The other gameObject isn't the player
- We are `inevent`
- We are in a `pause`
- `hit` is true 

The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true.

If we aren't in a `minipause`, a WaitForEvent coroutine starts which does the following:
- `hit` is set to true (this prevents a second trigger)
- We enter a `minipause`
- Yield frames until `switchcooldown` expires
- Yield another additional frame
- Call CancelAction on the player
- Yield a frame
- The [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[0]` is started with this object as the caller
- `hit` is set to false
- If `data[1]` isn't 0 or doesn't exist, the object is destroyed
