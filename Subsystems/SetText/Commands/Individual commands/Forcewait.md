# Forcewait

Yields for a certain amount of seconds if unpaused and stop a 
[Text advance](../../Related%20Systems/Text%20advance.md) skiptext if one was in progress.

## Syntax

````
|forcewait,seconds|
````

## Parameters

### `seconds`: float

The amount of seconds to yield for. This must be a valid float or an exception will be thrown.

## Remarks

If the pause menu is active, this command does nothing.

If honoring the skiptext is needed, use [Wait](Wait.md) instead.
