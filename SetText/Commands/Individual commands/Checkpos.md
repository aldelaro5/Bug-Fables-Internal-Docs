# Checkpos

Redirect to a different line by a [Dialogue line id](../Dialogue%20line%20id.md) if the current player's position on a specific axis isn't at least a specific value or do nothing otherwise.

## Syntax

````
|checkpos,axis,minvalue,redirect|
````

## Parameters

### `axis`: `x` | `y` | `z`

The axis to perform the position test. Any other value will cause the redirection to occur no matter the current position.

### `minvalue`: float

The minimal value the current position must have in the `axis` axis to not redirect. This must be a valid float or an exception will be thrown.

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) of the line to redirect if applicable. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

If the player is null, this command does nothing.

If the redirect happens, it is done using an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the resolved dialogue line and the command will cause processing to resume at the start of the new input string. Additionally, [Text advance](../../Related%20Systems/Text%20advance.md)'s skiptext will be disabled.
