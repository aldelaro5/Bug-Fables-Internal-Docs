# AddItem

Give an [Items](../../../Enums%20and%20IDs/Items.md) based on its id or a [flagvar](../../../Flags%20arrays/flagvar.md) containing it or transfer all the items selected to or from the storage if the last [ItemList](../../../ItemList/ItemList.md) had a multi selection.

## Syntax

(1)

````
|additem,itemtype,itemid|
````

(2)

````
|additem,itemtype,var,flagvar|
````

(3) (Not recommended, see remarks, only for transferring all multi selected items)

````
|additem|
````

## Parameters

### `itemtype`: `0` | `1` | `2`

The inventory type this command should give the [Items](../../../Enums%20and%20IDs/Items.md):

* 0: Standard items.
* 1: Key items. 
* 2: Storage items.

Any other value will cause an exception to be thrown.

### `var`

This is just what determines how the [Items](../../../Enums%20and%20IDs/Items.md) id is obtained. Any other value will this parameter to be interpreted as the `itemid`.

### `itemid`: int

The [Items](../../../Enums%20and%20IDs/Items.md) id to attempt to give. This must be a valid int value or an exception will be thrown.

### `flagvar`:  int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the item id. This must corresponds to an int of a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

If the last [ItemList](../../../ItemList/ItemList.md) was allowed to `multiselect` and a multi selection was performed, this overrides the behavior of this command to transfer to or from the storage depending on the [listtype](../../../ItemList/listtype.md). Specifically, this will take all the selection from `multiselect`, remove them from the list it came from and add them to the other based on the [listtype](../../../ItemList/listtype.md) that tells which one is the source. Every parameters gets ignored so it behaves like syntax (3).

Using syntax (3) directly is however not recommended. It can only work if `multiselect` was allowed and processed on the last [ItemList](../../../ItemList/ItemList.md). If it wasn't, this will throw an exception because there will be no check to determine that there are no parameters. It is best to send parameters in case no multi selection was performed.

The other syntax forms adds an item, but unlike [Additemtoss](Additemtoss.md), it will not check nor prompt for a toss. This means that it is possible to violate the constraint of having no more than the maximum amount of standard items or storage items by using this command. To check the quantity before giving the item, [Additemtoss](Additemtoss.md) or [Checkinvqtd](Checkinvqtd.md) are recommended, but it is always possible to manually perform this check in code.
