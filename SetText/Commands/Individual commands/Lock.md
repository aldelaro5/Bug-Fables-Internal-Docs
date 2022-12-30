# Lock

Force a grab on the [message](../../Global%20vars%20used/message.md) and the `minipause` locks.

## Syntax

````
|lock|
````

## Parameters

None

## Remarks

This command should only be used in [Dialogue mode](../../Dialogue%20mode.md) mode because the locks will only get released as part of the `dialogue cleanup phase`. That being said, these locks are normally grabbed in the `dialogue setup phase` so this command should only be used to regrab them if they were released during SetText.

This command is unused under normal gameplay.
