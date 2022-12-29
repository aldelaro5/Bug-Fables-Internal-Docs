# Caravanmedal

Process a [Goto](Goto.md) in syntax (2) if there are more than 1 prize [Medal](../../../Enums%20and%20IDs/Medal.md) set to be available through the Caravan shop or do nothing otherwise.

## Syntax

````
|caravanmedal,redirect|
````

## Parameters

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to send to [Goto](Goto.md) if applicable. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown if [Goto](Goto.md) is processed with it.

## Remarks

The [Goto](Goto.md) command is processed immediately if applicable.
