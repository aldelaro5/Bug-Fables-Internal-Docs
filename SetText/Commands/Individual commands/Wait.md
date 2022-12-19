# Wait

Yields for a certain amount of seconds if a [Text advance](../../Related%20Systems/Text%20advance.md) skiptext isn't in progress.

## Syntax

````
|forcewait,seconds|
````

## Parameters

### `seconds`: float

The amount of seconds to yield for. This must be a valid float or an exception will be thrown.

## Remarks

If a [Text advance](../../Related%20Systems/Text%20advance.md)'s skiptext was in progress, this command will do nothing. An exception to this is when in control of a [Minibubble](Minibubble.md) which will always yield regardless.

To override the skiptext, use [Forcewait](Forcewait.md) / [Fwait](Fwait.md) instead.
