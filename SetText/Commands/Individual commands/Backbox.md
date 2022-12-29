# Backbox

Render a box behind the text with 40% transparency using one of the text colors.

## Syntax

(1)

````
|backbox|
````

(2)

````
|backbox,colorindex|
````

## Parameters

### `colorindex`: int

The color index to render from the text color palette. This must be a valid color index or an exception will be thrown. Here are all the valid values:

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

The default is 4.

## Remarks

The backbox is a new UI object named `backbox` childed to the [textholder](../../Notable%20local%20variable/textholder.md) with its sortingOrder set to the current [Sort](Sort.md) - 2. It is set to cover the whole space of the [textholder](../../Notable%20local%20variable/textholder.md).

The color always ends up with 40% transparency no matter what the original color was.
