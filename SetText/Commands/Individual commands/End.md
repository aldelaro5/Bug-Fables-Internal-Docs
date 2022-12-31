# End

Signal to SetText to not wait for a confirmation input at the end after the input string has been fully processed in [Dialogue mode](../../Dialogue%20mode.md) mode

## Syntax

````
|end|
````

## Parameters

None

## Remarks

In [Dialogue mode](../../Dialogue%20mode.md) mode, SetText would normally yield control to the game by setting [waitinput](../../Global%20vars%20used/waitinput.md) to true in the [life cycle > Dialogue Cleanup](../../life%20cycle.md#dialogue-cleanup) phase. This command allows to bypass this behavior and prevents the yield entirely. Additionally, if the caller was an [Items](../../../Enums%20and%20IDs/Items.md), it will call Death on it. While it is generally preferred to place this command on the actual end of the line if the caller is an Item, as far as skipping the wait behavior, it does not matter where this command is placed. All it does is set the flag to skip it for later.

This commands is only applicable in [Dialogue mode](../../Dialogue%20mode.md) mode. It will not do anything in non dialogue mode.
