# Stars

Renders a specific amount of white stars sprites in a row.

## Syntax

````
|stars,amount|
````

## Parameters

### `amount`: int

The amount of white stars sprites to render. This must be a valid int or an exception will be thrown. Anything 0 or below will have this command do nothing.

## Remarks

The sprite is the id 100 from the gui's atlas. Each sprite is childed to the [textholder](../../Notable%20local%20variable/textholder.md) and rendered with the following properties:

* localScale: (0.5, 0.5, 0.5) or (0.5, 0.25, 0.5) when in a battle
* No angles
* White color
* layer: 5
* sortingOrder: [Sort](Sort.md) + an offset from 0 to `amount` - 1 depending on which one is rendered from the series (this makes them render on to of their left neighbor)
* localPosition: (offset, line, 1.0) where offset and line are the current offset and line position.

This command will increase the current offset by the horizontal sprite's extent - 0.15 for each one rendered.

It should be noted that this command does not work correctly under [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) because the offset calculations are off which means the stars may not render on the screen or it may render at the incorrect position.
