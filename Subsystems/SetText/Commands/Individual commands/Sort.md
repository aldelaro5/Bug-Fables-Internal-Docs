# Sort

Changes the sorting order of every letter rendered from now on.

## Syntax

````
|sort,sortingorder|
````

## Parameters

### `sortingorder`:  int

The sortingorder value to use. This value must be a valid int value or an exception will be thrown. Only values between -32768 and 32767 are accepted by Unity. Any value outside this range is undefined behaviour.

## Remarks

In [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md), this affects the whole line at once including letters rendered before this command on that line.
