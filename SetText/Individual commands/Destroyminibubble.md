# Destroyminibubble

Force the destruction of all or a specific [minibubble](Minibubble.md) by its index in the bubbles list if it wasn't destroyed yet.

## Syntax

(1) (This destroys all [minibubble](Minibubble.md) that aren't destroyed yet in the bubbles list)

````
|destroyminibubble|
````

(2)

````
|destroyminibubble,index|
````

## Parameters

### `index`: int

The index of the [minibubble](Minibubble.md) to destroy. This must be a valid int or an exception will be thrown. If the value is a valid int, but no element exists in the bubbles list at that index or the [minibubble](Minibubble.md) itself was already destroyed, this command will do nothing.

## Remarks

This command has mainly 2 use cases. It can not only preemptively destroy a minibubble, but if the minibubble was setup with an [halt](Halt.md) command as the inner SetText call, this command now controls when to destroy it.

For more details on how a minibubble destruction works, check [minibubble > Minibubble's destruction](Minibubble.md#minibubble-destruction), for destroying it from the same SetText call than a minibubble command, check [minibubble > Managing Minibbubles from the outer SetText call](Minibubble.md#managing-minibbubles-from-the-outer-settext-call) and for how to use it with [halt](Halt.md), check [minibubble > A note about Halt](Minibubble.md#a-note-about-halt).
