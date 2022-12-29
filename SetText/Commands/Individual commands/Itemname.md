# Itemname

Set a [flagstring](../../../Flags%20arrays/flagstring.md) to an [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md)'s name where the id is specified directly or contained in a [flagvar](../../../Flags%20arrays/flagvar.md).

## Syntax

(1)

````
|itemname,item|
````

(2)

````
|itemname,item,flagstring|
````

(3)

````
|itemname,medal,flagstring,1|
````

## Parameters

### `item`: int | `v`int | `var`int

The [Items](../../../Enums%20and%20IDs/Items.md) id to get the name of. The int form must be a valid [Items](../../../Enums%20and%20IDs/Items.md) id or an exception will be thrown. The `v`/`var` prefix indicates that the int portion is a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the id and it must be a valid one or an exception will be thrown.

### `flagstring`: int

The [flagstring](../../../Flags%20arrays/flagstring.md) slot to set to the `item`'s name. This must be a valid [flagstring](../../../Flags%20arrays/flagstring.md) slot or an exception will be thrown. The default value is 0.

### `medal`: int | `v`int | `var`int

The [Medal](../../../Enums%20and%20IDs/Medal.md) id to get the name of. The int form must be a valid [Medal](../../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown. The `v`/`var` prefix indicates that the int portion is a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the id and it must be a valid one or an exception will be thrown.

### `1`

Indicate to operate in syntax (3) which will set `flagstring` [flagstring](../../../Flags%20arrays/flagstring.md) slot to a [Medal](../../../Enums%20and%20IDs/Medal.md)'s instead of an [Items](../../../Enums%20and%20IDs/Items.md)'s name. This also honor MYSTERY? and will set the value to menu text 59 (?????) if [flags](../../../Flags%20arrays/flags.md) 681 is true. Any other value is ignores and will cause the command to operate in syntax (2).
