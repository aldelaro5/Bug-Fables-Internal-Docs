# Mothfly

Replace the text of this command to a series of a specific amount of |[Icon](Icon.md),146,`size`\| commands in a row where `size` is configurable.

## Syntax

(1)

````
|mothfly|
````

(2)

````
|mothfly,amount|
````

(3)

````
|mothfly,amount,size|
````

## Parameters

### `amount`: int

The amount of [Icon](Icon.md) commands to place in a row. This must be a valid int or an exception will be thrown. The default value is 1.

### `size`: float

The `size` parameter to send to each [Icon](Icon.md) command. This must be a valid float or an exception will be thrown. The default value is 0.35.

## Remarks

If there are less than 2 rendered [Icon](Icon.md) in [Dialogue mode](../../Dialogue%20mode.md), ` ` or any letter in [regular letter rendering](../../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md), this command will also append `  ` after the first occurrence and ` ` after the second one.

Char loop processing will resume at the same position than this command to accommodate the text replacement.
