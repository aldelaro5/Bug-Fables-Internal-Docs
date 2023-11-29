# ShopKeeper
Similar to [talk](Talk.md), but with a shop keeper buying or selling dialogue line.

> NOTE: This interaction is heavily involved in the [shop system](../Shop%20system.md) which are complex. It is only assigned at loading time from a [Shop](Shop.md) interaction and it should never be set in map entities game data. Read the shop system documentation to learn more.

## MapControl.CreateEntities
This interaction is heavily involved in the building of the shop structure. See the [shop system](../Shop%20system.md) documentation to learn more.

## args meaning
- `buy`: The input string will be overriden to `dialogues[6].y` (the shop keeper's buying [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md)) and the parent becomes the `shopkeeper` transform. This only happens during the course of a [Shop](Shop.md) interaction, but not in this interaction.

## SetUp
If `dialogues[10].x` floored is 1 (this is a medal shop), [SetBadgeShop](../Notable%20methods/SetBadgeShop.md) is called.

## Interact
The same as [talk](Talk.md), but the input string of the [SetText](../../../SetText/SetText.md) call is the one resolved from `dialogues[0].y` as the [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) which is shop keeper's selling line (or `dialogues[6].y` which is the buying one if `args` is `buy`). 

Also if `args` is `buy`, the parent of the SetText call is the `shopkeeper` and the `playerdata` entities will face it instead of this position.

## PlayerControl.LateUpdate
The same as [Talk](Talk.md).