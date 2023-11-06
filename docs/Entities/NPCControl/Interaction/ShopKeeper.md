# ShopKeeper
Similar to [talk](Talk.md), but with a shop keeper.

## args meaning
- `buy`: The input string will be overriden to `dialogues[6].y` and the parent becomes the `shopkeeper` transform.

## SetUp
If `dialogues[10].x` floored is 1, [SetBadgeShop](../Notable%20methods/SetBadgeShop.md) is called.

## Interact
Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being `dialogues[0].y` (or `dialogues[6].y` if `args` is `buy`):
- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- This transform as the parent (or the `shopkeeper` one if `args` is `buy`) and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) called on them with the position being this position and with forceback.