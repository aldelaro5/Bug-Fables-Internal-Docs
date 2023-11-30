# CaravanBadge
Similar than [Shop](Shop.md), but for interacting with a shelved Caravan prize medal which only involve some parts of the [shop system](../Shop%20system.md) because it is handled entirely on its own with its own set of data.

> NOTE: This interaction will cause the [entitytype](../NPCType.md) to be overriden to [SemiNPC](../Shop%20system.md#seminpc) which features the same logic than the ones used for the shop system.

## Data Arrays
- `data[0]`: An [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) corresponding to the shop keeper the interaction will target when Interact is called
- `data[1]`: The [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) of the shop keeper (whose entity id is `data[0]`) to call [SetText](../../../SetText/SetText.md) with when Interact is called

## args meaning
None.

## SetUp
The same than [Shop](Shop.md), but CaravanMedalSet features extended logic.

### CaravanMedalSet
The logic for this interaction changes as it doesn't follow the usual [Shop system](../Shop%20system.md) workflow:
- `shopkeeper` is set to the entity resolved using `data[0]` as the [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md)
- The array of the currently available prize medals at the caravan is obtained via PrizeBadges(true)
- If the array is empty, this object gets destroyed as there's no need to have it
- Otherwise:
  - The prize [medals](../../../Enums%20and%20IDs/Medal.md) array is shuffled
  - MainManager.`caravanorder` is set to the shuffled array
  - entity.`item` is set to true making it an [item entity](../../EntityControl/Item%20entity.md)
  - entity.`animid` is set to 2 (medal)
  - entity.`animstate` is set to the first element of the shuffled array (this is the first prize medal id of the array)
  - entity.`itemstate` is set to entity.`animstate`
  - [entitytype](../NPCType.md) is set to [SemiNPC](../Shop%20system.md#seminpc) which behaves the same than the ones used in the shop system
  - If we are rerolling (which only happens if we are calling from a [kill](../../../SetText/Individual%20commands/Kill.md) or [rerollshops](../../../SetText/Individual%20commands/Rerollshops.md) commands), DeathSmoke particles are played at this NPCControl position
  - The first element of MainManager.`caravanorder` is removed
- entity.`rigid` gets its gravity disabled with all constraints frozen
- The position is set to entity.`startpos`
- entity.`ccol` is disabled
- `descwindow` is destroyed if it existed
- The `scol` is disabled

## HasHiddenItem
The same than [Shop](Shop.md).

## Interact
- If this is an `item` entity, the entity `animstate` is set to its `itemstate`
- [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to the entity `animstate`
- [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to the buying price of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to 35
- [flagstring](../../../Flags%20arrays/flagstring.md) 0 to be the name of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to `menutext[59]` (?????)
- Call [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) using `data[1]` as the [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) for the input string (the shop keeper's caravan medal dialogue line) with the following:
  - [fonttype](../../../SetText/Notable%20states.md#fonttype) of `BubblegumSans`
  - `messagebreak` as the linebreak
  - No tridimensional
  - No position offset
  - No camera offset
  - size of Vector3.one
  - The parent is the entity resolved using `data[0]` (the shop keeper map entity id) as the [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md)
  - This NPCControl as the caller
- All `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md) call on them to face the entity resolved using `data[0]`(the shop keeper map entity id) as the [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md)

## SetBadgeShop
The logic changes drastically from [Shop](Shop.md) as the initialisation of the shelved item is delegated to CaravanMedalSet entirely which will be called instead of the logic of creating shelved items. If we are refreshing however, instance.`showmoney` is still set to 10.0 after which shows the berry count HUD element.

## LateUpdate (Every 3 frames, non `dummy` and the entity is `incamera`)
The same than [Shop](Shop.md), but this interaction always causes the instance.`showmoney` to be set to 1.0 before calling [CreateDescWindow](../Notable%20methods/CreateDescWindow.md) if we are calling it.

## PlayerControl.LateUpdate
The same than [Shop](Shop.md).

## EntityControl.[Unfix](../../EntityControl/EntityControl%20Methods.md#unfix)
The same than [Shop](Shop.md).

## [SetText](../../../SetText/SetText.md)
This interactions affects SetText in several ways, but all of them are the same than [Shop](Shop.md) except fot [kill](../../../SetText/Individual%20commands/Kill.md) which will cause SetBadgeShop with refresh to be called on the entity to kill (not its `shopkeeper`) if the caller has this interaction and the entity to kill's `animid` is 2 (meaning it's a [medal](../../../Enums%20and%20IDs/Medal.md) as it's assumed that this is an `item` entity). Additionally, this interaction also causes this command to skip calling [Death](../../EntityControl/Notable%20methods/Death.md) on the entity to kill.