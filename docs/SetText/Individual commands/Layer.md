# Layer

Changes the current layer to render from now on instead of the default layer which is 5 (`UI`).

## Syntax

````
|Layer,newlayer|
````

## Parameters

### `newlayer`: int

The new layer to render from now on. This must be a valid int or an exception will be thrown.

## Remarks

By default, all SetText calls renders to layer 5 (`UI`), but this command allows to change the layer to use. It affects the rendering of the following:

- All letters in [single letter rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)
- All [dropshadow](Dropshadow.md) in [single letter rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)
- All letters in [regular letter rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) when not using tridimensional
- All ButtonSprites used in the [Button](Button.md) command when not using tridimensional
