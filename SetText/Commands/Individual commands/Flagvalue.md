# Flagvalue

Replace the text of this command by the string representation of a [flags](../../../Flags%20arrays/flags.md) slot or one where the slot is contained in a [flagvar](../../../Flags%20arrays/flagvar.md) slot.

## Syntax

(1)

````
|flagvalue,flags|
````

(2)

````
|flagvalue,dummy,flagvar|
````

## Parameters

### `flags`: int

The [flags](../../../Flags%20arrays/flags.md) slot to print its the bool value's string representation. This must be a valid [flags](../../../Flags%20arrays/flags.md) slot or an exception will be thrown.

### `dummy`: var

This value is ignored, its presence is only to indicate to operate in syntax (2).

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot whose value is the [flags](../../../Flags%20arrays/flags.md) value's string representation to print. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

This command is intended for troubleshooting purposes. Its only usage is in the TestRoom.

This command resumes processing at the same position than this command to accommodate for the text replacement.
