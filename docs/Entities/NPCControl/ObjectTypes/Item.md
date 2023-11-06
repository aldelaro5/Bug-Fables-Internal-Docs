# Item
A collectable [item](../../../Enums%20and%20IDs/Items.md) or crystal berry bound to a [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot.

## Data Arrays
- `data[0]`: The [item](../../../Enums%20and%20IDs/Items.md) id this relates to (overriden to `data[3]` on SetUp)
- `data[1]`: ???
- `data[2]`: If it's anything other than 0, the item cannot be caught using the player `beemerang`
- `data[3]`: The [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot if applicable

## HasHiddenItem
If the entity's `animid` is 1 or 2, then `activationflag` must not be negative and the [flag](../../Flags%20arrays/flags.md) slot of that value must be false. 

If the `animid` is 3, then the [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) whose id is `data[0]` must not have been obtained yet

## CreateEntities
- The `animstate` is set to the `animid`
- The `animid` is set to `data[0]`
- `item` is set to true
- The `sprite` local position is set to (0.0, 0.5, 0.0)

## Setup
First, the layer is set to 0 (default).

Then, a few adjustements are performed on the entity:
- `onground` is set to false
- `alwaysactive` is set to true
- The `ccol` bounciness is set to 0.75

All collisions between the `ccol` and these entity's `ccol` are ignored:
- Objects with tag `PFollower`
- Any NPCControl (unless this is a `SemiNPC`)
- Any playerdata except the first one

Then, if the entity's `animid` is 3 (which means it's a crystal berry) and `tempobject` is false, then this is setup as a crystal berry. If it was obtained (the id is contained in `data[3]`), then the entity's `iskill` is set to true.Otherwise, adjustements on the entity are performed:
- `animstate` is set to the `regionaflag` ???
- The `sprite`'s local position is set to Vector3.zero
- AddModel is called with path `Prefabs/Objects/CrystalBerry` without offset
- `spin` is set to (0.0, 2.0, 0.0)
- `item` is set to true
- The `rigid`'s gravity is enabled

Finally, AddPlayerTrigger is called which will initialise `secondcoll` to a new trigger CapsuleCollider with the same properties than the entity's `ccol` and ignores all collisions between the player's `ccol` and the entity's `ccol`.

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerEnter if the other collider is the player
If we aren't in a `pause` or `minipause`, the `insideid` matches the current one and the `timer` is exactly -1.0 or above 1.0, a [CheckItem](../CheckItem.md) coroutine is started and `collisionammount` is incremented

## OnTriggerEnter main segment
The `beerang` is set to the other transform if all of the following are true:
- The other gameObject tag is `BeeRang`
- `data[2]` is either not present or it is and it's value is 0
- `beerang` was null
- We aren't in a `pause`, or `minipause` and [message](../../../SetText/Notable%20states.md#message) is released
- The player `beemerang` is present and its `hit` is false
- `insideid` matches the current one
