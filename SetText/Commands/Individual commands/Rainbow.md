# Rainbow

Toggles the rainbow [FontEffects](../../Related%20Systems/FontEffects.md) that will be effective from this point in [regular letter rendering](../../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md).

## Syntax

````
|rainbow|
````

## Parameters

None

## Remarks

This turns the rainbow [FontEffects](../../Related%20Systems/FontEffects.md) from false to true or from true to false depending on its existing state. The state change will affect every letter from the point this command is being processed.

This command does not work correctly in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md). While it will toggle the effect correctly on the letter slot, it will not look correctly.

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

This is one of the command supported by [Testdiag](Testdiag.md) if the command name part is written as `line` with case sensitivity.
