# Color

Set the text color to one of the game's text color palette for every letter from now on.

## Syntax

````
|color,colorindex|
````

## Parameters

### `colorindex`:  int

The color index to use from the game's text color palette. This value must be an int value between 0 and 9 or an exception will be thrown.

The colors available are:

* 0: 000000 (pure black, fully opaque).
* 1: EE0B0B (mostly red, fully opaque).
* 2: 00E700 (mostly green, fully opaque).
* 3: 0000FF (pure blue, fully opaque).
* 4: FFFFFF (pure white, fully opaque).
* 5: A9F1FF (sky blue, fully opaque).
* 6: FFD500 (mostly yellow, fully opaque).
* 7: 7C7C7C (gray, 214/255 opaque).
* 8: 00CC01 (mostly darker green than 2, fully opaque).
* 9: FFA400 (mostly orange, fully opaque).

## Remarks

This commands works in [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), but it will affect the whole line rather than each letter from now on.

This is one of the command supported by [Testdiag](Testdiag.md).
