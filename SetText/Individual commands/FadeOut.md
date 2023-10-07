# FadeOut

Play a fade out visual transition with options to specify the speed after a [fadein](FadeIn.md).

## Syntax

(1)

````
|fadeout|
````

(2)

````
|fadeout,speed|
````

(3) (while it technically works, this is the same then syntax (2) in practice)

````
|fadeout,speed,data|
````

(4) (while it technically works, this is the same then syntax (2) in practice)

````
|fadeout,speed,data,color|
````

## Parameters

### `speed`: float

The speed of the transition expressed in the desired percentage of color blending over the total amount of blending left to apply each frame. The speed is frame time independent, but the game will try its best to honor it. This must be a valid float or an exception will be thrown. The default value is 0.05.

### `data`: int

This parameter is parsed and sent to the transition coroutine, but is effectively not used. If specified, it still must be a valid int or an exception will be thrown. This can be ignored entirely.

### `color`: char

This parameter is parsed and sent to the transition coroutine, but it is effectively not used. This can be ignored entirely.

## Remarks

After the parameters are gathered, PlayTransition is called which first starts by stopping the previous transition coroutine if it exist. After that, the transition coroutine is started.

This coroutine will only do anything if there was an existing transition GameObject that was created during the previous [fadein](FadeIn.md). Consequently, this command will do nothing if a [fadein](FadeIn.md) wasn't performed before. The transition itself is done by doing a Lerp from the dimmer's color to a transparent color at percentage `speed` per framestep (which is 1 ideally, but this is adjusted based on the frame time). This is done until the color is fully transparent with a yield for a frame between each iteration. After the transition is complete, the dimmer GameObject is destroyed.

Additionally, if the transition takes at least 600 frames (about 10 seconds), a failsafe will trigger that will force the transition to stop.
