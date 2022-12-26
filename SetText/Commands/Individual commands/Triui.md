# Triui

Toggle the camera used to render each letters to 3DGUI instead of GUICamara (main camera in tridimensional) in [regular letter rendering](../../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md).

## Syntax

````
|triui|
````

## Parameters

None.

## Remarks

This command will function in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md), but its effect are not implemented with this method effectively doing nothing.

When toggled on, this will overwrite the layer that each letter renders at. Normally, it is rendered with the layer 5 (UI, uses GUICamera) or 0 (default, uses the main camera) in tridimensional, but this command, if enabled, will use layer 15 (3DUI) instead. This causes a different camera to render the text: the 3DGUI one instead of the regular GUICamera one and the main difference is it is in perspective mode rather than orthographic mode. This is needed to allow the depth of the text to look correct relative to where it is rendered in the world. The regular GUICamera cannot do this because it is in orthographic mode which renders everything at the same depth.

When toggled off, the text after will render using the GUICamera again.

TODO: why use this over tridimensional?
