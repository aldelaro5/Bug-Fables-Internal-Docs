# Shaky

Toggles the shaky [FontEffects](../../Related%20Systems/FontEffects.md) that will be effective from this point in [regular letter rendering](../../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md).

## Syntax

````
|shaky|
````

## Parameters

None

## Remarks

This turns the shaky [FontEffects](../../Related%20Systems/FontEffects.md) from false to true or from true to false depending on its existing state. The state change will affect every letter from the point this command is being processed.

This command does nothing in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md).

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

This is one of the command supported by [Testdiag](Testdiag.md).
