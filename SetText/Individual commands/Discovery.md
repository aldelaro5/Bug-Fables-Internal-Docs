# Discovery

Enable a [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) or enable all [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), [Bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md), [Recipes entry](../../Enums%20and%20IDs/librarystuff/Recipes%20entry.md), [Records entry](../../Enums%20and%20IDs/librarystuff/Records%20entry.md) and seen [Areas](../../Enums%20and%20IDs/librarystuff/Areas.md).

## Syntax

(1)

````
|discovery,discoveryid|
````

(2)

````
|discovery,all|
````

## Parameters

### `discoveryid`: int

The [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id to enable. This must be between 0 and 255 or an exception will be thrown.

### `all`

This indicates to operate in syntax (2) which will enable all `librarystuff` flags.

## Remarks

Syntax (1) accepts any valid [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) slot, even unused ones.

Syntax (2) enabled every single slot of every librarystuff flags including all the unused ones. This includes all [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), [Bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md), [Recipes entry](../../Enums%20and%20IDs/librarystuff/Recipes%20entry.md), [Records entry](../../Enums%20and%20IDs/librarystuff/Records%20entry.md) and seen [Areas](../../Enums%20and%20IDs/librarystuff/Areas.md).
