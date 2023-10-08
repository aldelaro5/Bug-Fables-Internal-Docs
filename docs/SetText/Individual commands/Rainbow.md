# Rainbow

Toggles the rainbow [FontEffects](../Related%20Systems/FontEffects.md) that will be effective from this point in [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md).

## Syntax

````
|rainbow|
````

## Parameters

None

## Remarks

This turns the rainbow [FontEffects](../Related%20Systems/FontEffects.md) from false to true or from true to false depending on its existing state. The state change will affect every letter from the point this command is being processed.

This command does not work correctly in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md). While it will toggle the effect correctly on the letter slot, it will not look correctly.

This command is accumulated for the [Backtracking](../Related%20Systems/Backtracking.md) system.

This is one of the command supported by [testdiag](Testdiag.md) if the command name part is written as `line` with case sensitivity.
