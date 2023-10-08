# Limit

Changes the [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s line break value.

## Syntax

````
|limit,value|
````

## Parameters

### `value`: float | `null`

The new value to set the line break to. This must be a valid float or `null` or an exception will be thrown. `null` means to remove the existing linebreak and set it to null.

## Remarks

This is essentially the exact same command than [setbreak](Setbreak.md), but without the ability to run [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) after the change.

It is preferred to send the desired line break value as a parameter to SetText from the start instead of using this command. While it is used in very specific scenarios in the game, it is redundant because it is simpler to pass the correct value at first.
