# Boxstyle

Sets the textbox's sprite or disable its rendering from now on in [Dialogue mode](../../Dialogue%20mode.md)

## Syntax

````
|boxstyle,styleindex|
````

## Parameters

### `styleindex`:  int

The textbox style index to use from now on:

* -1: Disables the rendering of the [textbox](../../Notable%20local%20variable/textbox.md)
* 0: The default rounded border style.
* 1: A jagged border rectangular style.
* 2: The same as style 1, but with rounded corners.
* 3: A brown paper shaped style.
* 4: A semi translucent gray rectangular style.
* 5: A stone looking style (this is used when obtaining artifacts under normal gameplay).
* 6: The same style than 2, but the borders are cyan and the inside is dark gray.
* 7: The same style than 6, but the borders are red.

Any other value will cause this command to do nothing.

## Remarks

If `styleindex` is -1, this command also sets the [tailtarget](../../Notable%20local%20variable/tailtarget.md) to null.

This command does nothing in non [Dialogue mode](../../Dialogue%20mode.md).
