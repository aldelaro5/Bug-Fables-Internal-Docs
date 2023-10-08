# Setbreak

Changes the [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s `linebreak` value with the option to run [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) again after the change.

## Syntax

(1)

````
|setbreak,linebreak|
````

(2)

````
|setbreak,linebreak,true|
````

## Parameters

### `linebreak`: float | `null`

The maximum width to use by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) from now on. The float form must be a valid float or an exception will be thrown. `null` allows to not have any line breaking occur, but this is incompatible with syntax (2) as it will throw an exception since it still expects the line break value to not be null.

### `true`

The presence of this parameter indicates to operate in syntax (2) which will run [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) after the new line break value has been set. Any other value is ignored and essentially is equivalent to syntax (1).

## Remarks

It is preferred to send the desired line break value as a parameter to SetText from the start instead of using this command. While it is used in very specific scenarios in the game, it is redundant because it is simpler to pass the correct value at first.
