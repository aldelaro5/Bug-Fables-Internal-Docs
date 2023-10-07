# Center

Toggle the text centering during rendering.

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

If specified, also go back to the first line and reposition the [textholder](../Notable%20states.md#textholder) to `position` or (-5.5, 0.9, 0.0) if it was not specified. Any value is accepted here as the value itself does not matter, only its presence.

## Remarks

This will toggle the centralize rendering from either false to true or true to false depending on its existing state. This rendering will reposition the [textholder](../Notable%20states.md#textholder) to the center in [Letter processing](../Life%20Cycle.md#letter-processing) during rendering each time. This allows the text to be rendered in the center of the parent.
