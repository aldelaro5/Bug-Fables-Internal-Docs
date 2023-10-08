# ShowItemList

A static method in MainManager. This method aims to display an [ItemList](ItemList.md) or refresh an existing one.

## Signature

````cs
public static void ShowItemList(int type, Vector2 position, bool showdescription, bool sell)
````

## Parameters

### `type`

The type of [ItemList](ItemList.md) to display. For more details, see [listtype](listtype.md).

### `position`

The position to render the [ItemList](ItemList.md) relative to GUICamera.

### `showdescription`

Whether to show the `listdescbox` of the selected option.

### `sell`

Whether to define the [ItemList](ItemList.md) as an item sell list. This influences the ability to multi select and the description rendering which will have a line detailing the selling price of the item hovered on when `showdescription` is true.

## Remarks

This can also be called via the [Pickitem](../SetText/Individual%20commands/Pickitem.md) [SetText](../SetText/SetText.md) command.

For more information about how ItemLists work, check these documents:

* [ItemList](ItemList.md)
* [ShowItemList Life Cycle](ShowItemList%20Life%20Cycle.md)
