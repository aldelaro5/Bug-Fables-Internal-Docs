# Boxstyle

Sets the textbox's sprite or disable its rendering from now on in [Dialogue mode](../Dialogue%20mode.md) and optionally specify its sortingOrder.

## Syntax

(1)
````
|boxstyle,styleindex|
````

(2)
````
|boxstyle,styleindex,sortorder|
````

## Parameters

### `styleindex`:  int

The textbox style index to use from now on:

* -1: Disables the rendering of the [textbox](../Notable%20states.md#textbox)
* 0: The default rounded border style.
* 1: A jagged border rectangular style.
* 2: The same as style 1, but with rounded corners.
* 3: A brown paper shaped style.
* 4: A semi translucent gray rectangular style.
* 5: A stone looking style (this is used when obtaining artifacts under normal gameplay).
* 6: The same style than 2, but the borders are cyan and the inside is dark gray.
* 7: The same style than 6, but the borders are red.

Any other value will cause this command to do nothing.

### `sortorder`: int

The new sortingOrder to set the SpriteRenderer of the textbox. This is optional: if this parameter isn't specified, the sortingOrder of the textbox isn't changed. If there is a value, this must be a valid int or an exception will be thrown.

## Remarks

If `styleindex` is -1, this command also sets the [tailtarget](../Notable%20states.md#tailtarget) to null.

This command does nothing in non [Dialogue mode](../Dialogue%20mode.md).
