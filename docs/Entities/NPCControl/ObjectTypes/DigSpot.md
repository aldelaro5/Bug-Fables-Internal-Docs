# DigSpot
A spot where an [item](../../../Enums%20and%20IDs/Items.md), [medal](../../../Enums%20and%20IDs/Medal.md) or Crystal Berry can be dug out or have an [event](../../../Enums%20and%20IDs/Events.md) be triggered when doing so using Kabbu's Dig ability.

## Data Arrays
- `data[0]`: This has 3 possible meanings:
  - If it's 1, it indicates that there is a crystal berry hidden in this spot
  - If it's 2 or above, it indicates that an [event](../../../Enums%20and%20IDs/Events.md) will start when the dig spot is dug up 
  - If it's 0 or less, it indicates that there is an [item](../../../Enums%20and%20IDs/Items.md) or [medal](../../../Enums%20and%20IDs/Medal.md) hidden in this spot
- `data[1]`: The meaning depends on `data[0]`:
  - If `data[0]` is 2 or above, the [event](../../../Enums%20and%20IDs/Events.md) id that will start when the dig spot is dug up
  - If `data[0]` is 1, the [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot hidden in this spot
  - If `data[0]` is 0 or less, the item type hidden in this spot
- `data[2]`: If `data[0]` is 0 or less, the [item](../../../Enums%20and%20IDs/Items.md) or [medal](../../../Enums%20and%20IDs/Medal.md) id hidden inside this spot. This can be considered mandatory if `data[0]` isn't 1 because it is still expected to be present even if `data[0]` is 2 which doesn't make use of this field. It is optional if `data[0]` is 1 as the value is ignored

## Additional data
- `regionalflag`: If it's positive and `data[0]` is 0 or less, it's the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot that is bound to the created item when the dig spot is dug up.
- `activationflag`: If it's above 0 and `data[0]` is 0 or less, the [flag](../../../Flags%20arrays/flags.md) slot that is bound to the created item when the dig spot is dug up

## HasHiddenItem conditions
The condition depends on `data[0]`.

If `data[0]` is 1, the [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) whose id is `data[1]` must not have been obtained yet. 

If `data[0]` isn't 1, then `data[1]` must be 1 (key item) and `data[2]` must be 52 (Lore Book).

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

Then, if `data[0]` is 1 and the corresponding [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot in `data[1]` has already been obtained, the entity's `iskill` is set to true which ends this object setup as the dig spot will not appear.

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
This does nothing if any of the following are true:
- The entity.`iskill` is true
- The other gameObject isn't the player
- The player `uproot` is false (meaning the player wasn't preparing to go back to the surface after stopping to dig)

The `DigPop2` sound is played following by `DigPop`, but with 0.7 volume.

If `data[0]` is 2 or above, a WaitForEvent coroutine starts which does the following:
- `hit` is set to true
- We enter a `minipause`
- Yield frames until `switchcooldown` expires
- Yield another additional frame
- Call CancelAction on the player
- Yield a frame
- The [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[1]` is started with this object as the caller
- `hit` is set to false

Otherwise (meaning there is an item or a Crystal Berry hidden in this spot):
- CreateItem is called with:
  - This object's position as the startpos, 
  - If `data[0]` is 1, 3 (Crystal Berry) is the itemtype, it's `data[1]` otherwise
  - If `data[0]` is 1, `data[1]` is the itemid (which is a [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot), it's `data[2]` otherwise
  - RandomItemBounce(4.0, 15.0) as the direction
  - If `data[0]` is 0 or less (there is an item or medal hidden in this spot), the timer is 300.0 if `data[1]` is 0 (standard item). It is -1 (no timer) otherwise
- If `data[0]` is 0 or less, the `activationflag` and `regionalflag` of the created item are set to the corresponding ones from this object
- All collisions between the entity and the created item's entity are ignored

Finally, the entity `iskill` is set to true which prevents to undig the spot again.
