# PressurePlate
A plate that can be pushed down in its receptacle using a [PushRock](PushRock.md) or optionally, the player or an NPCControl entity.`icecube`. This can optionally start an [event](../../../Enums%20and%20IDs/Events.md) when actuated for the first time.

## Data Arrays
- `data[0]`: If 1, the player is allowed to actuate the plate
- `data[1]`: If 1, an NPCControl entity.`icecube` can actuate the plate
- `data[2]`: An [event](../../../Enums%20and%20IDs/Events.md) id whose event will start when the plate is actuated for the first time
- `vectordata[0]`: the offset to displace the plate when it is pressed relative to its receptacle

## Additional Data
- `activationflag`: If it's not -1, it's the [flag](../../../Flags%20arrays/flags.md) slot that if true, will set `hit` to true and not allow any deactuation. Anything lower than -1 is considered invalid and will result with the plate being stuck as deactuated even when pressing on it.

## Setup
`moveobj` is set to the first child of the entity's `model` which is the actual plate of the model inside its receptacle unless the entity.`originalid` is the `TestButton` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) where the `moveobj` is set to the entity.`sprite`. 

If the entity.`originalid` is `AncientPressurePlate`, a GlowTrigger is added on that plate object setting this NPCControl as the `parent` and the `glowparts` to one element being the plate's MeshRenderer.

If the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, the `hit` is set to true and the `moveobj`'s local position to `vectordata[0]` which sets the plate as pressed and visually renders it so.

Finally, a few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
We first needs to determine if `hit` should be set to true or false.

If the `activationflag` is not negative and its [flag](../../../Flags%20arrays/flags.md) slot is true, then `hit` is set to true. Otherwise, it is set to true until the `actioncooldown` expires at which point it is set to false. The `actioncooldown` is decremented by the game's framestep whenever it is set to true in this case.

From there, if `hit` is true, the `moveobj` local position is set to a lerp from the existing one to `vectordata[0]` with a factor of a 1/5 of the game's frametime. If `hit` is false, it's the same lerp, but the to vector is Vector3.zero instead. This basically smoothly displaces the plate when `hit` is true and pulls it back to its starting position when `hit` is false.

## OnTriggerStay
This does nothing if any of these are true:
- The other gameObject is the player `beemerang`
- The `activationflag` (if it's not -1) refers to a [flag](../../../Flags%20arrays/flags.md) slot whose value is true
- All of the following are false:
  - The other gameObject's Hornable doesn't exist (meaning this isn't an entity.`icecube`)
  - The other gameObject tag is `PushRock`
  - `data[0]` is 1 and the other gameObject is the player
  - `data[1]` is 1 and the other gameObject is an NPCControl with its `icecube` not null

- If `hit` is false, [SwitchSound(true)](../Notable%20methods/SwitchSound.md) is called
- The other gameObject's Hornable `onground` is set to true (the entity.`icecube`)
- If `data[2]` exists and isn't negative, the [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[2]` is started with this object as the caller and the value is set to -1 after the start which prevents the event to be started again
- `hit` is set to true
- `actioncooldown` is set to 10.0

## OnTriggerExit
This does nothing if the other gameObject is the player `beemerang`, or if none of the following are true:
- The other gameObject has a Hornable (it's an entity.`icecube`)
- The other gameObject tag is `PushRock`
- `data[1]` is 1 and the other gameObject has an NPCControl
- `data[0]` is 1 and the other gameObject is the player

The following occurs if the conditions above are met:
- `hit` is set to false
- `actioncooldown` is set to 0.0
- [SwitchSound(false)](../Notable%20methods/SwitchSound.md) is called