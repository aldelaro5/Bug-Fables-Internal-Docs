# Checkminibubble

Redirect if any [minibubble](Minibubble.md) are present to a different dialogue line or do nothing if none are present.

## Syntax

````
|checkminibubble,lineid|
````

## Parameters

### `lineid`: int

The [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect if applicable. This must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

This command was specifically made to allow the Gen & Eri switch puzzle to work inside the Honey Factory.
