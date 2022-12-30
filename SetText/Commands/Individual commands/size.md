# Size

Set the text size scaling to render from now on with the ability to prevent processing of further size commands temporarily.

## Syntax

(1)

````
|size,sizeuniform|
````

(2)

````
|size,sizex,sizey|
````

(3)

````
|size,unpause,sizex,sizey|
````

(4)

````
|size,sizex,sizey,lockcontrol|
````

## Parameters

### `sizeuniform`: float

The text size scaling to set uniformly in the vertical and horizontal direction. This must be a valid float or an exception will be thrown.

### `sizex`: float | `x`

The text size scaling to set horizontally. The float form must be a valid float or an exception will be thrown. `x` will cause the existing y size (NOT X SIZE which is likely an error!) to be used instead. Any other value will be interpreted as the float form which will cause an exception to be thrown if it isn't one.

### `sizey`: float | `y`

The text size scaling to set vertically. The float form must be a valid float or an exception will be thrown. `x` will cause the existing y size to be used instead which essentially ignores the value. Any other value will be interpreted as the float form which will cause an exception to be thrown if it isn't one.

### `unpause`

Tells this command to only change the size when unpaused and inlist from the [ItemList State Machine](../../../ItemList/ItemList%20State%20Machine.md) is true (NOTE: see [inlist issue](../../../ItemList/inlist%20issue.md) for potential issues). If paused or inlist is false, the size will not change, but the rest of the command will still be processed (see remarks for details). Any other value will process the command in syntax (2) which will throw an exception.

### `lockcontrol`: `lock` | `force` | `unlock`

Tells how to treat this command in regards to the size locks which tells SetText to not process further size commands:

* `lock`: Engages the size lock. Any further size commands not sent with this parameter set to `force` or `unlock` will do nothing.
* `force`: Ignores the size lock when processing the command, but do not disengage the size lock. Only this command will process normally.
* `unlock`: Disengages the size lock. Further size commands will process normally.
  Any other value is ignored.

## Remarks

In [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), this also sets the localScale of the letter slot to (size.x, size.y, 1.0) * 0.07. This essentially scales everything down to 7% of the new size.

Additionally, the bleep volume is adjusted to the new size's magnitude which will make bleeps go louder or softer depending on the new size.

This command affects [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by changing the horizontal size scaling used to predict the amount of space further letters will take. This is done ignoring the locking system so even if the command isn't going to change the size once processed, it will still be taken as a width change for the auto line breaker which can cause problems.

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system, but it does so by adding the same command twice in a row. (NOTE: Why?)
