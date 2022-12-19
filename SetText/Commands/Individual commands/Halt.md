# Halt

Completely stall SetText via an infinite loop that yields a frame.

## Syntax

````
|halt|
````

## Parameters

None.

## Remarks

This command will provoke an infinite stall by yielding every frames from now on. There are no way to stop it unless the entire SetText coroutine call is saved into a reference and StopCoroutine is called on it or if the GameObject that started it is destroyed.

On [Dialogue mode](../../Dialogue%20mode.md), this will completely softlock the game because of the message lock while on non [dialogue mode](../../Dialogue%20mode.md), the call will linger forever, but so long as no locks were acquired before this command, the game should still function.

This command has a special use case in conjunction with the [Minibubble](Minibubble.md) and [Destroyminibubble](Destroyminibubble.md) commands which when used together can allow to decide when to destroy any Minibubble if this command is sent at the end of the inner SetText call of the Minibubble. For more details, check [Minibubble > A note about Halt](Minibubble.md#a-note-about-halt) from the [Minibubble](Minibubble.md) command page.
