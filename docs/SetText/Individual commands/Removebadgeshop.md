# Removebadgeshop

Removes a [medal](../../Enums%20and%20IDs/Medal.md) from a medal shop by its [medal](../../Enums%20and%20IDs/Medal.md) id or by a [flagvar](../../Flags%20arrays/flagvar.md) containing it.

## Syntax

(1)

````
|removebadgeshop,medalshopid,medalid|
````

(2)

````
|removebadgeshop,medalshopid,var,flagvar|
````

## Parameters

### `medalshopid`: int

The medal shop id to remove the [medal](../../Enums%20and%20IDs/Medal.md) from. This must be a valid medal shop id or an exception will be thrown.

Here are the different values available:

|Id|Name|
|--:|----|
|0|Merab's shop|
|1|Shades's shop|

### `medalid`: int

The [Medal](../../Enums%20and%20IDs/Medal.md) to remove from the medal shop. This must be a valid int or an exception will be thrown.

### `var`

The presence of this is only to indicate to operate in syntax (1). Any other value will be interpreted as `medalid` which may cause an exception to be thrown.

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot containing the [Medal](../../Enums%20and%20IDs/Medal.md) to remove from the medal shop. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

If the [Medal](../../Enums%20and%20IDs/Medal.md) does not exist in the given shop, this command does nothing.

If the concerned medal shop is Shades's shop, this will also increment [flagvar](../../Flags%20arrays/flagvar.md) 66 with the assumption that the medal was bought before the removal.
