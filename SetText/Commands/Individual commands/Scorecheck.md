# Scorecheck

Render the Termacade's games high scores and yield until Confirm or Cancel is pressed.

## Syntax

````
|scorecheck|
````

## Parameters

None.

## Remarks

This command will do the following:

* Start a fade to black and yield for one second
* Remove the camera positional limits by settings the bounds from (-999.0, -999.0, -999.0) to (999.0, 999.0, 999.0) and save the existing ones in the temporary fields.
* Set the camera to move instantaneously to (0.0, 70.0, 0.0).
* Save the state of the anti aliasing and then force enable it.
* Set the render texture of the camera with down sampling of 80%.
* Setup a solid color object named `score` whose color is 75% between white and black at 0.01 pixels per unit positioned at (0.0, 70.0, 0.0) with the pivot set at the center (0.5, 0.5).
* Call SetText in non [Dialogue mode](../../Dialogue%20mode.md) where the input string is |[center](Center.md)\||[color](Color.md),4||[dropshadow](Dropshadow.md),0.1,-0.1||[textangle](Textangle.md)\| followed by menu text 237 (|[rainbow](Rainbow.md)\|!! HIGH SCORES !!) where the parent is `score`:
  * [fonttype](../../fonttype.md) is `D3Streetism`
  * Rendered using `tridimensional` at true
  * Position is (0.0, 4.8, 0.0)
  * Camera offset is Vector3.one
  * Size is (2.0, 2.0)
  * No caller
* Create a new GameObject and child `score` to it.
* Start a fade back to the game and yield for one second
* Render each game score like the following:
  * Call SetText in non [Dialogue mode](../../Dialogue%20mode.md) where the input string is |[center](Center.md)\||[color](Color.md),4||[dropshadow](Dropshadow.md),0.1,-0.1||[textangle](Textangle.md)\| followed by |[Center](Center.md)\| followed by menu text 210 for Flower Journey and 211 for Mite Knight (it contains the name of the game) where the parent is `score`:
    * [fonttype](../../fonttype.md) is `D3Streetism`
    * Rendered using `tridimensional` at true
    * Position is (-7.0, `y`, -3.0) where `y` is 2.0 for Flowey Journey and 0.35 for Mite Knight
    * Camera offset is Vector3.one
    * Size is (1.5, 2.0)
    * No caller
  * Yield for 0.5 seconds.
  * Call SetText in non [Dialogue mode](../../Dialogue%20mode.md) where the input string is |[center](Center.md)\||[color](Color.md),4||[dropshadow](Dropshadow.md),0.1,-0.1||[textangle](Textangle.md)\| followed by |[Center](Center.md)\| followed by the string representation of [flagvar](../../../Flags%20arrays/flagvar.md) 28 for Flower Journey or 29 for Mite Knight padded by 5 to the left with `0` (each contains the high score of the game) where the parent is `score`:
    * [fonttype](../../fonttype.md) is `D3Streetism`
    * Rendered using `tridimensional` at true
    * Position is (4.0, `y`, -3.0) where `y` is 2.0 for Flowey Journey and 0.35 for Mite Knight
    * Camera offset is Vector3.one
    * Size is (2.0, 2.0)
    * No caller
  * Yield for 0.5 seconds.
* Call SetText in non [Dialogue mode](../../Dialogue%20mode.md) where the input string is |[center](Center.md)\||[color](Color.md),4||[dropshadow](Dropshadow.md),0.1,-0.1||[textangle](Textangle.md)\| followed by menu text 238 (Press |[button](Button.md),4| or |[button](Button.md),5| to exit.) where the parent is the unnamed GameObject created earlier:
  * [fonttype](../../fonttype.md) is `D3Streetism`
  * Rendered using `tridimensional` at true
  * Position is (0.0, 0.0, -3.0)
  * Camera offset is Vector3.one
  * Size is (1.5, 2.0)
  * No caller
* Yield for 0.5 seconds.
* While Confirm or Cancel aren't pressed:
  * Set the position of the unnamed GameObject to (0.0, `y`, 0.0) where `y` is -2.0 if Mathf.Sin of Time.time is above 0 or 999.0 otherwise (this makes it oscillate between being visible or not by rendering the object way above the camera).
  * Yield for a frame.
* Once the yielding is done, make the bottom prompt disappear by moving it to (0.0, 999.0, 0.0).
* Start a fade to black and yield for a second.
* Restore the render texture to not have any down sampling.
* Restore the camera's anti aliasing state.
* Destroy `score`.
* Restore the camera positional limits using the temporary fields saved earlier.
* Reset the camera instantly.
* Yield for a frame.
* Fade back to the game.
* Yield for a second.
