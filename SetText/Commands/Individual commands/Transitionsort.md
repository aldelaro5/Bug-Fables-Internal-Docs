# Transitionsort

Change the sortingOrder of the current sprite used for transitions.

## Syntax

````
|transitionsort,sort|
````

## Parameters

### `sort`: int

The sortingOrder to set the transition sprite to. This must be a valid int or an exception will be thrown.

## Remarks

This command does nothing if there are not transition sprites.

This command is used in a very specific case where a [Minibubble](Minibubble.md) needed to be rendered over a [FadeIn](FadeIn.md) transition.
