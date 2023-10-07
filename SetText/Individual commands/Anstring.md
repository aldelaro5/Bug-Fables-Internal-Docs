# Anstring

Replaces the text from this command to a [medal](../../Enums%20and%20IDs/Medal.md) or [item](../../Enums%20and%20IDs/Items.md)'s prepender string from the game data.

## Syntax

(1)

````
|anstring,item|
````

(2)

````
|anstring,medal,ismedal|
````

## Parameters

### `item`: int | `var`int

The [Item](../../Enums%20and%20IDs/Items.md) to obtain the prepend string from. The int form must be a valid [Items](../../Enums%20and%20IDs/Items.md) id or an exception will be thrown. The `var`int form must have an int portion be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot containing a valid [Item](../../Enums%20and%20IDs/Items.md) is or an exception will be thrown.

### `medal`: int | `var`int

The [Medal](../../Enums%20and%20IDs/Medal.md) to obtain the prepend string from. The int form must be a valid [Medal](../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown. The `var`int form must have an int portion be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot containing a valid [Medal](../../Enums%20and%20IDs/Medal.md) is or an exception will be thrown.

### `ismedal`: int

When a non zero value is specified, syntax (2) is used. If the value is 0 or not specified, syntax (1) is used. This must corresponds to a valid int value or an exception will be thrown.

## Remarks

This command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
