# [Items](../../Enums%20and%20IDs/Items.md) list type

Display the list of items, key items or storage items.

## Options generation

`listvar` will be the list of [items](../../Enums%20and%20IDs/Items.md) in that specific item type or a single element with -1 as its value if there is no items.

## Option's SetText input string

If the [items](../../Enums%20and%20IDs/Items.md) id in question is -1, the string will be |[color](../../SetText/Commands/Individual%20commands/Color.md),1| followed by the no items message from MenuText line id 20.

Otherwise, a sprite of the item matching its id will be rendered mostly on the left side of the bar. Its localScale is 0.55, 0.6, 1.0 when unpaused and 0.9, 0.9, 1.0 when paused. The string will be the item name from itemdata.

When paused, the bar height used for calculating the y position of the down arrow is overridden to -0.9.

## Description box rendering

Not rendered if the item id is -1.

If it isn't, It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the item's description obtained from itemdata using the item id. The secondary string used if `sell` is true is the item's name.

## Confirmation handling (when unpaused)

First, [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the item id selected. If the item id selected is -1, immediately destroy the list which ends this list's processing.

Otherwise, the [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the selected item's name from itemdata. If `listsell`, the behaviour depends if we were using `multiselect`:

* If we are, [flagstring](../../Flags%20arrays/flagstring.md) 0 is overridden to the MenuText line id 279 (which is "Lot of Items" in English), [flagvar](../../Flags%20arrays/flagvar.md) 10 is now the sum of each selected item's buying price / 2 floored clamped from 1 to 999. [Flags](../../Flags%20arrays/flags.md) 349 is now true
* If we aren't, [flagvar](../../Flags%20arrays/flagvar.md) 10 is the buying price of the selected item / 2 floored clamped from 1 to 999.
  At any case, the list gets destroyed afterwards which ends this list's processing.

## Other behaviors (when unpaused)

If this is a storage items list, `multiselect` is allowed. If it is a standard items list, [flags](../../Flags%20arrays/flags.md) 349 has to be true to allow multi selection. Adding an item to the selection is restricted by not exceeding the current maximum amount of standard items for storage list or the maximum amount of storage items for items list (this maximum is always 35 under normal gameplay). A buzzer sound will play if adding an item to the selection would violate this restriction.
