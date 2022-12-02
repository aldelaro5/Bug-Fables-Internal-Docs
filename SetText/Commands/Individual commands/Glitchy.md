# Glitchy

Toggles the glitchy [FontEffects](../../Related%20Systems/FontEffects.md) that will be effective from this point in non [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md).

## Syntax

(1)

````
|glitchy|
````

(2)

````
|glitchy,1|
````

## Parameters

### `superglitch`: `1`

Sets this effect to use the `superglitch` version which is the more aggressive version of glitchy. Any value other than `1` will be ignored and the behavior will be like (1).

## Remarks

This turns the glitchy [FontEffects](../../Related%20Systems/FontEffects.md) from false to true or from true to false depending on its existing state. The state change will affect every letter from the point this command is being processed. 

This command does nothing in [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md).

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

This is one of the command supported by [Testdiag](Testdiag.md).
