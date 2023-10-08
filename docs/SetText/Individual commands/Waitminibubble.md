# Waitminibubble

Yield until all or a specific [minibubble](Minibubble.md) by its index in the bubbles list is destroyed.

## Syntax

(1) (This yields until all [minibubble](Minibubble.md) are destroyed in the bubbles list)

````
|waitminibubble|
````

(2)

````
|waitminibubble,index|
````

## Parameters

### `index`: int

The index of the [minibubble](Minibubble.md) to yield until its destruction. This must be a valid index in the bubbles list or an exception will be thrown.

## Remarks

In syntax (1), this will loop through the list until there are no elements in it. In each iteration, if the [minibubble](Minibubble.md) got destroyed, it is removed from the bubbles list before the yield for a frame is performed which removes its consideration in the test to know if we are done.

For more details, check [Minibubble > Managing Minibbubles from the outer SetText call](Minibubble.md#managing-minibbubles-from-the-outer-settext-call).
