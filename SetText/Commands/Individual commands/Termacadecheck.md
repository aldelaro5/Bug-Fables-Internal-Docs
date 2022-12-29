# Termacadecheck

A [Goto](Goto.md) helper where the [Dialogue line id](../Dialogue%20line%20id.md) can be one of 2 values depending if both high scores at the Termacade games were beaten or not from their default one.

## Syntax

````
|termacadecheck,notbeatenline,beatenline|
````

## Parameters

### `notbeatenline`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to send to [Goto](Goto.md) if any of the high scores are not higher than their default one. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown if [Goto](Goto.md) is processed with it.

### `beatenline`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to send to [Goto](Goto.md) if both high scores are higher than their default one. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown if [Goto](Goto.md) is processed with it.

## Remarks

The [Goto](Goto.md) command is immediately processed with the resolved line in syntax (1) depending on [flagvar](../../../Flags%20arrays/flagvar.md) 28 (Mite Knight's high score) being strictly higher than 9500 or not and [flagvar](../../../Flags%20arrays/flagvar.md) 29 (Flower Journey's high score) being strictly higher than 4500 or not.
