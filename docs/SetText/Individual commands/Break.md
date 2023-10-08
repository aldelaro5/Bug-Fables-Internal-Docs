# Break

Wait for a confirmation input before resuming.

## Syntax

````
|break|
````

## Parameters

None

## Remarks

This behaves the same way than [next](Next.md) except it does not record the text for [Backtracking](../Related%20Systems/Backtracking.md) and it does not process a [blank](Blank.md) command after. It only stops the [tailtarget](../Notable%20states.md#tailtarget) from talking in [Dialogue mode](../Dialogue%20mode.md) mode and yields to the game using [waitinput](../Notable%20states.md#waitinput) as well as reset the [Text advance](../Related%20Systems/Text%20advance.md) text skip values.
