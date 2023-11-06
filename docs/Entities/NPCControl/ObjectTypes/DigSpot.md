# DigSpot
A spot where something can be dug out using Kabbu's Dig ???

## Data Arrays
- `data[0]`: If it's 1, it indicates that there is a crystal berry related to this object. There's no crystal berry associated if it has any other value.
- `data[1]`: The [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot associated with this object if `data[0]` is 1. NOTE: this must be a valid crystalbfflags slot or an exception will be thrown if `data[0]` is 1.
- `data[2]`: ???

## HasHiddenItem conditions
If `data[0]` is 1, the [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) whose id is `data[1]` must not have been obtained yet. If this is false or doesn't apply, then `data[1]` must be 1 and `data[2]` must be 52 for this to still have a hidden item.

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

Then, if `data[1]` is applicable and the crystal berry corresponding to its id has been obtained, the entity's `iskill` is set to true which ends this object setup as the grass will not appear.

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
This does nothing if any of the following are true:
- The entity `iskill` is true
- The other ghameObject isn't the player
- The player `uproot` is false

First, the `DigPop2` sound is played following by `DigPop`, but with 0.7 volume.

If `data[0]` is 2 or above, a WaitForEvent coroutine starts which does the following:
- `hit` is set to true
- We enter a `minipause`
- Yield frames until `switchcooldown` expires
- Yield another additional frame
- Call CancelAction on the player
- Yield a frame
- The [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[1]` is started with this object as the caller
- `hit` is set to false

Otherwise:
- [CreateItem](../CreateItem.md) is called with `base.transform.position, (data[0] != 1) ? data[1] : 3, (data[0] != 1) ? data[2] : data[1], MainManager.RandomItemBounce(4f, 15f), (data[0] != 1) ? GetItemTimer(data[1]) : (-1)` TODO ???
- If `data[0]` is 1, the `data` of the item created earlier is overwritten completely to be a new array of a single element being `data[1]`. Otherwise, the `activationflag` and `regionalflag` of the created item are set to the corresponding ones from this object
- All collisions between the entity and the created item's entity are ignored

Finally, the entity `iskill` is set to true
