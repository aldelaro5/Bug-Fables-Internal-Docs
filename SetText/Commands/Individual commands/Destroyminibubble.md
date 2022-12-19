# Destroyminibubble

Force the destruction of all or a specific [Minibubble](Minibubble.md) by its index in the bubbles list if it wasn't destroyed yet.

## Syntax

(1) (This destroys all [Minibubble](Minibubble.md) that aren't destroyed yet in the bubbles list)

````
|destroyminibubble|
````

(2)

````
|destroyminibubble,index|
````

## Parameters

### `index`: int

The index of the [Minibubble](Minibubble.md) to destroy. This must be a valid int or an exception will be thrown. If the value is a valid int, but no element exists in the bubbles list at that index or the [Minibubble](Minibubble.md) itself was already destroyed, this command will do nothing.

## Remarks

This command has mainly 2 use cases. It can not only preemptively destroy a [Minibubble](Minibubble.md), but if the [Minibubble](Minibubble.md) was setup with an [Halt](Halt.md) command as the inner SetText call, this command now controls when to destroy it.

For more details on how a [Minibubble](Minibubble.md) destruction works, check [Minibubble > Minibubble's destruction](Minibubble.md#minibubble-s-destruction), for destroying it from the same SetText call than a [Minibubble](Minibubble.md) command, check [Minibubble > Managing Minibbubles from the outer SetText call](Minibubble.md#managing-minibbubles-from-the-outer-settext-call) and for how to use it with [Halt](Halt.md), check [Minibubble > A note about Halt](Minibubble.md#a-note-about-halt).
