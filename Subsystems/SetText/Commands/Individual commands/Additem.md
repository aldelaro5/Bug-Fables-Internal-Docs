# AddItem

Give an item based on its id or a flagvar containing it or transfer all the items selected to or from the storage.

## Syntax

(1)

````
|additem|
````

(2)

````
|additem,itemtype,itemid|
````

(3)

````
|additem,itemtype,var,flagvar|
````

## Parameters

### `itemtype`: `0` | `1` | `2`

The inventory type this command should give the item:

* 0: Standard items.
* 1: Key items. 
* 2: Storage items.

Any other value will cause an exception to be thrown.

### `var`

This is just what determines how the item id is obtained. Any other value will this parameter to be interpreted as the `itemid`.

### `itemid`: int

The item id to attempt to give. This must be a valid int value or an exception will be thrown.

### `flagvar`:  int

The flagvar slot containing the item id. This must corresponds to an int of a valid flagvar slot or an exception will be thrown.

## Remarks

Syntax (1) is special since it specifically operates after the selection of items from a storage withdraw or deposit operation. It will take all the selection from `multiselect`, remove them from the list it came from and add them to the other based on the [listtype](../../../ItemList/listtype.md) that tells which one is the source.

The other syntax forms adds an item, but unlike [Additemtoss](Additemtoss.md), it will not check nor prompt for a toss. This means that it is possible to violate the constraint of having no more than the maximum amount of standard items or storage items by using this command. To check the quantity before giving the item, [Additemtoss](Additemtoss.md) or [Checkinvqtd](Checkinvqtd.md) are recommended, but it is always possible to manually perform this check in code.
