# FadeIn

Play a fade in visual transition with options to specify the speed, color and the ability to destroy the caller after a [fadeout](FadeOut.md) done right after this one.

## Syntax

(1)

````
|fadein|
````

(2)

````
|fadein,speed|
````

(3)

````
|fadein,speed,data|
````

(4)

````
|fadein,speed,data,color|
````

(5)

````
|fadein,speed,data,kill|
````

## Parameters

### `speed`: float

The speed of the transition expressed in the desired percentage of color blending over the total amount of blending left to apply each frame. The speed is frame time independent, but the game will try its best to honor it. This must be a valid float or an exception will be thrown. The default value is 0.05.

### `data`: int

This parameter is parsed and sent to the transition coroutine, but is effectively not used. It still must be a valid int or an exception will be thrown.

### `color`: char

The color index to use from the game's text color palette for the fade in. This value must be an int value between `0` and `9` or the value will be ignored. If a string is specified, only the first char counts. The default value is 0 or pure black.

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

### `kill`

This parameter's presence instructs the command to operate using syntax (5) which also plays a fade out transition after the fade in and destroy the caller between the transitions. Any other value is ignored.

## Remarks

After the parameters are gathered, PlayTransition is called which first starts by stopping the previous transition coroutine if it exist as well as destroy all GameObjects used during said transition. After that, the transition coroutine is started.

This Coroutine first creates a GameObject called "Dimmer" with a SpriteRender bound to a texture with a uniform transparent color. This dimmer is then set to cover the whole screen with the layer set to 5, the parent to the GUICamera, the sortingOrder to 9999 and the local position to (0.0, 0.0, 10.0) without angles. The transition itself is done by doing a Lerp from the texture's color to the target color at percentage `speed` per framestep (which is 1 ideally, but this is adjusted based on the frame time). This is done until the color is fully opaque with a yield for a frame between each iteration.

Additionally, if the transition takes at least 600 frames (about 10 seconds), a failsafe will trigger that will force the transition to stop.

As for using syntax (5), this will follow up with a special version of [fadeout](FadeOut.md). First, the caller is set to not be talking. Then, a yield of 0.75 + (1 - `speed`) seconds is performed (which means the default is 1.7 seconds). The caller is then destroyed and PlayTransition is called with a [fadeout](FadeOut.md) instead using the same parameters than the one sent to this command. Finally, a second yield is done whose time is identical to the one performed before the [fadeout](FadeOut.md). While the [fadeout](FadeOut.md) itself isn't a command getting processed, the actual transition is performed in an equivalent way semantically.

If syntax (5) is not used, a [fadeout](FadeOut.md) has to happen after this command to be able to see the screen. The transition is only visual in nature.
