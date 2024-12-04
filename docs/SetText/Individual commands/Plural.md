# Plural

Replaces this command's text by one of 2 strings depending on if a value or a [flagvar](../../Flags%20arrays/flagvar.md) slot containing it is equal to 1 or not.

## Syntax

````
|plural,value,onestring,notonestring|
````

## Parameters

### `value`: int | `var`int | `v`int | `money`

The value to check for equality to 1 or the [flagvar](../../Flags%20arrays/flagvar.md) slot containing it. The int form be a valid index of the stat bonuses list or an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `onestring`: string
The text to replace this command's text by if `value` is equal to 1.

### `notonestring`: string
The text to replace this command's text by if `value` is NOT equal to 1.

## Remarks

Processing will resume at the same position than this command to accommodate the text replacement.

This command is unused under normal gameplay, but remains functional.
