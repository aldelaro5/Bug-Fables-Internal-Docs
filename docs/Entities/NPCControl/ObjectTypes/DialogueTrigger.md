# DialogueTrigger
A zone where a [SetText](../../../SetText/SetText.md) call starts in [dialogue mode](../../../SetText/Dialogue%20mode.md) using a map dialogue line id. It can either unconditionally trigger as a one shot call on the first Update cycle or trigger via OnTriggerStay that starts the call when the player collides with it with the option to destroy the NPCControl after.

## Data Arrays
- `data[0]`: The map dialogue line id to call [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) when triggered
- `data[1]`: If not 0 or doesn't exist, the object is destroyed after triggering it, only applicable if `data[2]` doesn't exist or isn't 1
- `data[2]`: If 1, the SetText call is unconditionally started on the first Update cycle before destroying this NPCControl making it a one shot call. This is optional, if it doesn't exist, this feature is disabled.

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

If the above is fufilled:

- CancelAction is called on the player 
- [SetText](../../../SetText/SetText.md) is called in [dialogue mode](../../../SetText/Dialogue%20mode.md) using the map dialogue line id `data[0]`:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - Standard messagebreak as `linebreak`
    - No `tridimensional`
    - No local `position` change, no `camoffset` change or `size` change
    - This transform as the `parent`
    - No `caller`

If the current map supports the [global commands](../../../SetText/Related%20Systems/GlobalCommand.md) system, the map's `currentline` is set to `data[0]`.

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
- Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being the one resolved by the [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) contained in `data[0]`:
    - `BubblegumSans` as the [fonttype](../../../SetText/Notable%20states.md#fonttype)
    - The default `messagebreak` as the linebreak
    - No tridimensional
    - No position and camera offsets
    - Size of Vector3.one
    - This as the parent
    - No caller
- `hit` is set to false
- If `data[1]` isn't 0 or doesn't exist, the object is destroyed
