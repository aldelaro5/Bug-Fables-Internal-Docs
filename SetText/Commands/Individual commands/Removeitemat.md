# Removeitemat

Remove an [Items](../../../Enums%20and%20IDs/Items.md) by its inventory type and its index in that inventory which may come from [ItemList](../../../ItemList/ItemList.md)'s `listoption`.

## Syntax

(1) (Remove the item at [ItemList](../../../ItemList/ItemList.md)'s `listoption`)

````
|removeitemat,type|
````

(2)

````
|removeitemat,type,index|
````

## Parameters

### `type`: int

The inventory type of the [Items](../../../Enums%20and%20IDs/Items.md) to remove. This must be a valid inventory type or an exception will be thrown. The valid values are 0 for standard items, 1 for key items and 2 for stored items.

### `index`: int | `v`int | `var`int | `opt`

The index to remove the item at or a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing it. The int form must be a valid index in the `type` inventory type array or an exception will be thrown. The `v`/`var` prefix means the int porition is a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the index and it must be a valid one or an exception will be thrown. `opt` is equivalent to syntax (1) which will use [ItemList](../../../ItemList/ItemList.md)'s `listoption` instead.

## Remarks

If the [ItemList](../../../ItemList/ItemList.md) is in multi selection, this command does nothing.

Using syntax (1) implies that an [ItemList](../../../ItemList/ItemList.md) has been confirmed in single selection and this command is used as handler.
