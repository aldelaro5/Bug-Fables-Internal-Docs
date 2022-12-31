# Lock

Force a grab on the [message](../../Global%20vars%20used/message.md) and the `minipause` locks.

## Syntax

````
|lock|
````

## Parameters

None

## Remarks

This command should only be used in [Dialogue mode](../../Dialogue%20mode.md) mode because the locks will only get released as part of the [SetText Life Cycle > Dialogue Cleanup](../../SetText%20Life%20Cycle.md#dialogue-cleanup). That being said, these locks are normally grabbed in the [SetText Life Cycle > Dialogue setup](../../SetText%20Life%20Cycle.md#dialogue-setup) so this command should only be used to regrab them if they were released during SetText.

This command is unused under normal gameplay.
