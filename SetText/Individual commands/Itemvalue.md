# Itemvalue

Set a [flagvar](../../Flags%20arrays/flagvar.md) to an [Items](../../Enums%20and%20IDs/Items.md) or [Medal](../../Enums%20and%20IDs/Medal.md)'s value (buying price) where the id is specified directly or contained in a [flagvar](../../Flags%20arrays/flagvar.md).

## Syntax

(1)

````
|itemvalue,item|
````

(2)

````
|itemvalue,item,flagvar|
````

(3)

````
|itemvalue,medal,flagvar,1|
````

## Parameters

### `item`: int | `v`int | `var`int

The [Item](../../Enums%20and%20IDs/Items.md) id to get the value of. The int form must be a valid [Item](../../Enums%20and%20IDs/Items.md) id or an exception will be thrown. The `v`/`var` prefix indicates that the int portion is a [flagvar](../../Flags%20arrays/flagvar.md) slot containing the id and it must be a valid one or an exception will be thrown.

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot to set to the `item`'s value. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. The default value is 0.

### `medal`: int | `v`int | `var`int

The [Medal](../../Enums%20and%20IDs/Medal.md) id to get the value of. The int form must be a valid [Medal](../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown. The `v`/`var` prefix indicates that the int portion is a [flagvar](../../Flags%20arrays/flagvar.md) slot containing the id and it must be a valid one or an exception will be thrown.

### `1`

Indicate to operate in syntax (3) which will set `flagvar` [flagvar](../../Flags%20arrays/flagvar.md) slot to a [Medal](../../Enums%20and%20IDs/Medal.md)'s instead of an [Item](../../Enums%20and%20IDs/Items.md)'s name. This also honor MYSTERY? and will set the value to 35 if [flag](../../Flags%20arrays/flags.md) 681 is true. Any other value is ignores and will cause the command to operate in syntax (2).
