# DialogueTrigger
A dialogue trigger zone ???

## Data Arrays
- `data[0]`: The map dialogue line id to call [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) when entered ???
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

This update immediately and unconditionally (TODO: what ???) calls CancelAction on the player and calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) using the map dialogue line id `data[0]`:
- [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
- Standard messagebreak as `linebreak`
- No `tridimensional`
- No local `position` change, no `camoffset` change or `size` change
- This transform as the `parent`
- No `caller`

If the current map supports the [global commands](../../../SetText/Related%20Systems/GlobalCommand.md) system, the map's `currentline` is set to `data[0]`.

Finally, this object gets destroyed.

TODO: There's clearly more to this because it doesn't make sense to unconditionally start the SetText call... ???

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
