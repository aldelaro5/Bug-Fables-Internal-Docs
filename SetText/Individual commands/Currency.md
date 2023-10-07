# Currency

Replaces this command's text with a textual representation of a number followed by a space followed by either the singular form of the game's currency (berry in English) or the plural form (berries in English) depending on a specific amount or an amount contained in a [flagvar](../../Flags%20arrays/flagvar.md).

## Syntax

(1)

````
|currency,amount|
````

(2)

````
|currency,var,flagvar|
````

## Parameters

### `amount`: int

The amount of berries to test the plurality. This must be a valid int value or an exception will be thrown.

### `var`

This is just what determines how the amount of berries to test the plurality is obtained. Any other value will this parameter to be interpreted as the `amount`.

### `flagvar`:  int

The [flagvar](../../Flags%20arrays/flagvar.md) slot containing the amount of berries to test for the plurality. This must corresponds to an int of a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

More specifically, this command expects line 0 of MenuText to contain the singular form and line 1 of MenuText to contain the plural form.

This command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
