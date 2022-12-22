# Center

Toggle the `centralize` rendering method.

## Syntax

(1)

````
|center|
````

(2)

````
|center,reset|
````

## Parameters

### `reset`:  var

If specified, also go back to the first line and reposition the [textholder](../../Notable%20local%20variable/textholder.md) to [position](position.md) or (-5.5, 0.9, 0.0) if it was not specified. Any value is accepted here as the value itself does not matter, only its presence.

## Remarks

This will toggle the `centralize` rendering from either false to true or true to false depending on its existing state. This rendering will reposition the [textholder](../../Notable%20local%20variable/textholder.md) to the center in [life cycle > Letter processing](../../life%20cycle.md#letter-processing) each time. This allows the text to be rendered in the center of the [Parent](Parent.md).

This seems to be broken in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) TODO: why?
