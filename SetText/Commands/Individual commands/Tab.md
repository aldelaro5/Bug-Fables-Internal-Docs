# Tab

Set the tabsize of the current letter slot in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md)

## Syntax

````
|tab,tabsize|
````

## Parameters

### `tabsize`: int

The amount of spaces a `\t` character corresponds to. This must be a valid int or an exception will be thrown.

## Remarks

This command while functional doesn't do anything practical in [regular letter rendering](../../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md) because it only affects the letter slot used in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md).
