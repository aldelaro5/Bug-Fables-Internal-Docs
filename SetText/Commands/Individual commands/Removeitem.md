# Removeitem

Remove a [medal](../../../Enums%20and%20IDs/Medal.md) by its id or an [items](../../../Enums%20and%20IDs/Items.md) by its index from a specific inventory type where the item index or medal id is specified directly or is contained into a specified [flagvar](../../../Flags%20arrays/flagvar.md) slot. This can also remove all items that was multi selected in the last [ItemList](../../../ItemList/ItemList.md) if it was a `listsell`.

## Syntax

(1)

````
|removeitem,itemtype,item|
````

(2)

````
|removeitem,itemtype,var,itemflagvar|
````

(3) (Not recommended, see remarks, only for removing all multi selected items)

````
|removeitem|
````

## Parameters

### `itemtype`: `0` | `1` | `2` | `3`

The inventory type this command should give the item:

* 0: Standard items.
* 1: Key items. 
* 2: Storage items.
* 3: Medals (will cause `item`/`itemflagvar` to resolve to a medal id instead of an item index).

Any other value will cause an exception to be thrown.

### `var`

This value instructs the parser to use syntax (2). If `itemtype` is `3`, any other value will cause this parameter to be interpreted as `item` while if `itemtype` is NOT `3`, any value not containing `var` will cause this parameter to be interpreted as `item`.

### `item`: int

The [items](../../../Enums%20and%20IDs/Items.md) index or the [medal](../../../Enums%20and%20IDs/Medal.md) id to remove. 

For an item index, this must be a non negative number lower than the number of items in possession of the `itemtype` inventory type or an exception will be thrown.

For a medal id, this must be a valid integer or an exception will be thrown.

### `itemflagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot to get the [items](../../../Enums%20and%20IDs/Items.md) index or the [medal](../../../Enums%20and%20IDs/Medal.md) id to remove. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. After the value was obtained from the [flagvar](../../../Flags%20arrays/flagvar.md), the restrictions from `item` applies the same way.

## Remarks

This command will first check if `multiselect` was allowed and performed on the last [ItemList](../../../ItemList/ItemList.md) that was processed. If it was, this completely ignores any parameters which makes it behave like syntax (3) where it will remove all items that was selected from the list (`multiselect` already contains the options indexes which should already match the inventory indexes). If `multiselect` was allowed and performed, but `listsell` is false, this command will do nothing. Otherwise, this command proceeds as normal to remove the item or medal.

Using syntax (3) directly is however not recommended. It can only work if `multiselect` was allowed and processed on the last [ItemList](../../../ItemList/ItemList.md). If it wasn't, this will throw an exception because there will be no check to determine that there are no parameters. It is best to send parameters in case no multi selection was performed.
