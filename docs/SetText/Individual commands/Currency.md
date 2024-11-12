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

### Russian specific grammar support
If [languageid](../languageid.md) is 6 (`Russian`), the logic of what menutext line to use changes because the Russian language has very specific rules relating to plurality associated with specific numbers.

Here's a table that explains the menutext line used depending on the effective amount (line 276 is unused and is `ягода`):

|Effective amount|menutext line used|
|----------------|------------|
|Any that ends with `11`, `12`, `13` or `14`|275 (`ягод`)|
|Any that ends with `1`, but doesn't end with `11`|277 (`ягоду`)|
|Any that ends with `2`, `3` or `4` but doesn't end with `12`, `13` or `14`|278 (`ягоды`)|
|Any that ends with `5`, `6`, `7`, `8` or `9`|275 (`ягод`)|

It should be noted that in context, the command assumes grammatically that the number refers to a currency cost. This explains why menutext 277 (`ягоду`) is used over menutext 276 (`ягода`) because the former is used to describe cost. In practice, this matches most of the time where this command is used so it is the best compromise to assume the context of cost.
