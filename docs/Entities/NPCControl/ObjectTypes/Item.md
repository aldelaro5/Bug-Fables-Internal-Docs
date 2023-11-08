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

If it's an `item` (which should always be the case with this object type), the `colliderheight` is set to 1.0 and the entity's `ccol` center to be (0.0, 0.5, 0.0). This reduces the height of the collider to more snuggly fit the item shape. It is possible for this logic to apply outside of this object type, but only if it's used as a shelved shop item.

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## Non trapped active item update logic
Technically, this section overrides the main active one only when entity.`item` is true while active and not `trapped`, but effectively, it can only have a meaningful influence with this object type. The only other way to get into this section is to have an [item entity](../../EntityControl/Item%20entity.md) used as a shelved shop item, but when this happens, it doesn't do anything meaningful besides calling [StopForceMove](../../EntityControl/EntityControl%20Methods.md#StopForceMove) on the 3rd cycle onward effectively making it fixed in place.

If the `objecttype` is [Item](ObjectTypes/Item.md) and `beerang` isn't null, the following occurs:
- The `timer` is set to 300.0 if it was -1
- The absolute position is set to the `beerang` one + Vector3.Up
- The entity's Unifx gets called if it was a `fixedentity`

If the `secondcoll` is present (which is a trigger collider with the same properties than the `ccol`) then all collisions gets ignored between it and the player's `ccol` if the player is present and the `touchcooldown` hasn't expired yet. On the other hand, if the `touchcooldown` isn't exactly -9999.0 (TODO: what does this mean???), the collisions gets unignored and `toochcooldown` gets set to -9999.0. If neither happened, but the the `interacttype` isn't `Shop`, then the `ccol` is enabled. TODO: revisit this, it might be a debounce measure when tossing an item, but it's unclear ???

If the entity's `onground` is true, [Jump](../EntityControl/EntityControl%20Methods.md#jump) is called on it with the absolute value of its current `rigid`'s y velocity. It will also increment `bounces` if it hasn't reached 3 yet on top of playing the `"ItemBounce" + ((entity.animid != 3) ? "0" : "1"` (TODO: ???) sound effect if the `interacttype` isn't `Shop` and the `startlife` is above 15 frames.

If by there, `bounces` has reached 3, [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity.

If the entity's `sprite` is present, then the `timer` logic proceeds:
- If the `timer` has yet to expire and we aren't in a `minipause` or `pause`, the timer is decremented according to the frametime, but it is clamped from 0.0 to infinity. Otherwise, if the `timer` expired, this GameObject gets destroyed.
- If the `timer` is between -1.0 and 100.0 exclusive and we aren't in a `minipause` or `inevent`, the entity's `sprite.enabled` gets toggled. Otherwise, the entity's `sprite` is enabled. This logic blinks the sprite when less than 100 frames are left on the `timer`.

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
