# Shop system
The way the game defines a shop is complex and it involves a unique way to load its data and to create the actual shop at loading time. This results in multiple NPCControl being involved at runtime that forms the logic of the shop.

There are 2 types of shops:

- A standard [items](../../Enums%20and%20IDs/Items.md) shop (key items aren't supported, see how `dialogues[10].x` works below for details)
- A [medals](../../Enums%20and%20IDs/Medal.md) shop

Both are similar with only some logic differences. Notably, they both behave differently at loading time vs runtime. This means that the data defined in [map entities data](../../TextAsset%20Data/Entity%20data.md#`entitydata`%20directory) gets converted to different entities set at runtime. It involves both MapControl.CreateEntities and NPCControl.Setup.

There is technically a third type of shop involving the caravan prize medals, but this one only has one shelved item and its behavior is very different from the shop system this page will talk about even if some logic are shared. For more details on how caravan prize medals work, check the [CaravanBadge](Interaction/CaravanBadge.md) interaction page.

This page will describe the loading time behaviors as the runtime will be described in the different [Interactions](Interaction.md) involved.

## SemiNPC
This [NPCType](NPCType.md) is very special in the shop system because it is the game's only usage of them (there is no map entity data that loads it as it is considered invalid since the shop system created them when needed). Its functioning is essential for the shop system, enough to document it in this page.

The basic idea of a SemiNPC is it behaves mostly like an [Object](NPCType.md#object), but it allows interactions. This is ideal for a shelved item because it's possible to leave the [objecttype](Object.md#objecttypes) to `None` which means that by default, there is no logic that will exist on the entity (objects are normally entirely driven by their type so leaving to `None` means the least amount of logic occurs). This is what differentiate an [Item](ObjectTypes/Item.md) NPCControl which is a collectable [item entity](../EntityControl/Item%20entity.md) to an item SemiNPC which has no logic other than being a simple item entity that just happens to support interactions thanks to `SemiNPC` allowing it. Everything other than being interactable is either excluded from the NPCControl logic or is left practicaly useless. Effectively, it's a "dumb" item: it can only be interacted with when the player gets close enough, but nothing more.

Additionall, this type has no OnTriggerEnter logic. This type shouldn't ever be loaded from map entity data because the shop system will create them when needed.

## Map entity data
A shop is declared in the map entity data as a single entity with the [Shop](Interaction/Shop.md) interaction. That entity data will be transformed to the entire shop structure at loading time. This means that this interaction is only used for loading purposes here: it will not actually involve the entity having a shop interaction as that will be delegated to each shelved items.

The entity itself becomes the [ShopKeeper](Interaction/ShopKeeper.md) of the shop which happens by changing the `interacttype` on it. There will also be shelved items that gets created as `SemiNPC` with the [Shop](Interaction/Shop.md) interactions using the properties contained in the various data arrays of the shop keeper entity. The shelved items gets linked to their new shop keeper via the `shopkeeper` field.

For an item shop, most of this happens during MapControl.CreateEntities while for a medal shop, it's mostly done in NPCControl.SetUp.

One additional thing to point out about `dialogues`: this array for a shop ends up either empty or of length 20. If the length of the array in the shop entity isn't 0, then the actual value is ignored and it will always be of length 20. This forces the game to load all the `dialogues` data elements that can exists. If the length is 0 however, none are loaded and the array remains empty. This however will cause unexpected behaviors and it is therefore not recommended to do so.

Here is how the data of the shop is defined on the main shop entity:

- `dialogues[0].y`: The [dialogue line id](../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) of the shop keeper when interacting with them (managed by the [ShopKeeper](Interaction/ShopKeeper.md) interaction)
- `dialogues[1].y`: If it's 1, the shop accepts Crystal Berries instead of regular berries. NOTE: this won't behave as expected for a standard items shop, but it is technically supported by the game's data
- `dialogues[2].y`: For a standard items shop, if it's higher than 0.1, the `mmulti` field (the buying price multiplier) of each shelved item will be this value / 10.0 (the result of the multiplication will be ceiled). This is read, but not used for a medals shop
- `dialogues[6].y`: The [dialogue line id](../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) of the shop keeper when interacting with a shelved item
- `dialogues[8].x`: The `radius` of each shelved item will this value / 10.0 (defaults to 1.5625  if it's lower than 0.1)
- `dialogues[9].x`: For a medals shop, the medal shop id (only 0 and 1 exists under normal gameplay). This isn't used for a standard items shop
- `dialogues[10].x`: This is 0 for all standard item shops and 1 for a medals shop (it is not possible to specify anything else which prevents key items from being supported)
- `data.length`: The amount of shelved items
- `data`: For a standard items shop, the list of [item](../../Enums%20and%20IDs/Items.md) ids that the shop sells. This isn't used for a medals shop because the game manages them using the [shop available and pools arrays](../../External%20data%20format/Save%20File.md#line-4-array-line-medals-available-in-medal-shops) (the length is still used for the amount though)
- `vectordata`: The list of the shelved items position where each index matches the corresponding `data` element

## Standard items shop specifics
Here is how a shelved shop item gets created for each of them in MapControl.CreateEntities (all data pulled are from the shop keeper entity data):

- A new entity is created via [CreateNewEntity](../EntityControl/EntityControl%20Creation.md#createnewentity) named `FixedshopX` where X is the `data` index
- The `startpos` gets set to the corresonding `vectordata` element
- The `animid` gets set to `dialogues[10].x` (in practice however, this can only work with 0 as 1 will make it treat like a medals shop while 2 is invalid as it's a standard items shop, not a medals one)
- If `animid` is 0 (which it should always be), the `animstate` is set to the corresponding `data` element
- `item` is set to true making this an [item entity](../EntityControl/Item%20entity.md)
- `hasshadow` is set to false
- The NPCControl is added and set to `npcdata`
- npcdata.[entitytype](NPCType.md) is set to `SemiNPC`
- npcdata.[interacttype](Interaction.md) is set to [Shop](Interaction/Shop.md)
- `emoticonoffset` is set to (0.0, -1000.0, 0.0)
- npcdata.`shopkeeper` is set to the shop keeper's `npcdata`
- npcdata.`radius` is set to a 1/10 of `dialogues[8].x`, but if it ends up lower than 0.1, it defaults to 1.5625
- npcdata.`insideid` is set to the shop keeper's `insideid`
- `colliderheight` is set to 0.5

On SetUp, there's 3 logic that will inevitably follow thanks to the `Shop` interaction and the `shopkeeper` being set for each shelved items:

- `mmulti` is set to a 1/10 of `shopkeeper.dialogues[2].y`, but if it's less than 0.1, it defaults to 1.0. This will later get used to multiply the item's price and the result will be ceiled.
- CaravanMedalSet(false) is called which does the following:
    - entity.`rigid` gets its gravity disabled with all constraints frozen
    - The position is set to entity.`startpos`
    - entity.`ccol` is disabled
    - `descwindow` is destroyed if it existed
    - The `scol` is disabled

## Medals shop specifics
For this shop type, the only logic MapControl.CreateEntities does is set the the [interacttype](Interaction.md) of the original shop entity to [ShopKeeper](Interaction/ShopKeeper.md). All initialisation occurs in SeUp in 2 phases, one of the shop keeper and then one for each shelved items. The shelved items are initialised in the first phase then configured further on the second phase.

### Shop initialisation from the shop keeper's SetUp
For the first phase, thanks to `dialogues[10].x` being 1 and the [ShopKeeper](Interaction/ShopKeeper.md) interactions, SetBadgeShop(false) is called.

The first thing it does is initialises `shopitems` to a new list of EntityControl of length `data.length`. This is used to keep references of the shelved items entities in case the method is called with refresh which causes the destructions of the shelved items before recreating them

From there, a loop is done from 0 to `data.length` exclusive to actually create the shelved items (the `data` array is never indexed directly):

- A new entity is created with name `badgeshopX` where X is the current index
- `startpos` is set to the corresponding `vectordata`
- `animid` is set to 2 (medal)
- If the current index doesn't exist in `avaliablebadgepool` of the store id contained in `dialogues[9].x` or if it does exist, but it's -1, `iskill` is set to true. Otherwise, `animstate` is set to the corresponding medal id found in `avaliablebadgepool`
- `item` is set to true making it an [item entity](../EntityControl/Item%20entity.md)
- `hasshadow` is set to false
- The NPCControl is added and set to `npcdata`
- npcdata.[entitytype](NPCType.md) is set to `SemiNPC`
- npcdata.[interacttype](Interaction.md) is set to [Shop](Interaction/Shop.md)
- `emoticonoffset` is set to (0.0, -1000.0, 0.0)
- npcdata.`shopkeeper` is set to the current NPCControl (which is the shop keeper)
- npcdata.`radius` is set to a 1/10 of `dialogues[8].x`, but if it ends up lower than 0.1, it defaults to 1.5625
- npcdata.`insideid` is set to the shop keeper's `insideid`
- `colliderheight` is set to 0.5
- The new entity gets added to the map.`entities` and childed to the map
- The corresponding `shopitems` gets set to the new entity

### Initialising the shelved shop items
The second phase involves each shelved items on SetUp:

- `mmulti` is set to a 1/10 of `shopkeeper.dialogues[2].y`, but if it's less than 0.1, it defaults to 1.0. NOTE: this doesn't end up doing anything for a medals shop
- CaravanMedalSet(false) is called which does the following:
    - entity.`rigid` gets its gravity disabled with all constraints frozen
    - The position is set to entity.`startpos`
    - entity.`ccol` is disabled
    - `descwindow` is destroyed if it existed
    - The `scol` is disabled

### Refresh process
Medals shop have a unique capability: it's possible for the game to recreate the entire shop by passing true instead of false to SetBadgeShop. This is notably used during a [kill](../../SetText/Individual%20commands/Kill.md) or [rerollshops](../../SetText/Individual%20commands/Rerollshops.md) SetText commands.

Doing so will destroy all `shopitems` and their `descwindow` if it existed followed by a MainManager.UpdateShops call before recreating the shelved items. The UpdateShops will randomly sort all `availablebadgepool` arrays, not just the one the game wanted to refresh (this changes the items on the shelf because only the first ones are going to be present and the overflow ones discarded due to `iskill` being true).

From there, the shelved items are initialised as normal with one extra step: if the entity isn't `iskill` by the end, DeathSmoke particles will play on the entity's `startpos`

Finally, instance.`showmoney` is set to 10.0 which reveals the berry count HUD element.