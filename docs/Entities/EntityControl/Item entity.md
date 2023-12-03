# Item entity
An item entity is an entity that has its `item` field set to true. It completely changes the rendering and updates of that entity.

The summary as far as the fields are concerned:
- `item` is true
- `animid` is the item type, not an [animid](../../Enums%20and%20IDs/AnimIDs.md)
- `animstate` and `itemstate` are both the item id, the former is not an [animstate](Animations/animstate.md) and the latter is assigned on [Start](Start.md) and is used in [UpdateItem](Update%20process/UpdateItem.md)

Additionally, [UpdateItem](Update%20process/UpdateItem.md) is now part of the update process of this entity.

## Item types
The following is the values `animid` can have when we are dealing with an `item` entity. The value also dictates what `animstate` and `itemstate` mean.
|Value|`itemstate` meaning|Description|
|----:|------------------|----------|
|0|An [item](../../Enums%20and%20IDs/Items.md) id|Standard item|
|1|An [item](../../Enums%20and%20IDs/Items.md) id|Key item|
|2|A [medal](../../Enums%20and%20IDs/Items.md) id|Medal|
|3|A [crystalbfflag](../../Enums%20and%20IDs/crystalbfflags.md) id|Cystal Berry|

## How UpdateItem works
In practice, both the `itemstate` should always have the same value than the `animstate` because on [Start](Start.md), this is enforced by setting `itemstate` to `animstate` when `item` is true. On several (but not all) occasions, the caller also sets the `basestate` to the same value. This is relatively safe because it prevents the entity `animstate` to revert to the to default `Idle` animstate.

All of this allows a dedicated update method called [UpdateItem](Update%20process/UpdateItem.md) to resolve the correct `sprite` or to disable it in the case of a Crystal Berry (since only a model is rendered).

## Additional effects
Item entities have their own special logic outside of EntityControl. This section will detail them.

### MainManager.RefreshEntities
All item entities gets [UpdateItem](Update%20process/UpdateItem.md) called on them.

### MainManager.FixEntities
Item entities are exempt from the logic of this method which mean they don't get fixed in place.

### Hazards.OnTriggerEnter
If the other entity has no `npcdata` or there's one, but its `beerang` is null (it's not caught in the player beemerang) while `animid` is above 0 (it's not a standard item) or `animstate` isn't 6 or 7 (`MoneySmall` or `MoneyMedium`), ReturnEntity is called. NOTE: this doesn't exclude `MoneyBig` which seems like a bug as it can cause the destruction of the item when it wasn't intended.