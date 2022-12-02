# Break

Wait for a confirmation input before resuming.

## Syntax

````
|break|
````

## Parameters

None

## Remarks

This behaves the same way than [Next](Next.md) except it does not record the text for [Backtracking](../../Related%20Systems/Backtracking.md) and it does not process a [Blank](Blank.md) command after. It only stops the [tailtarget](../../Notable%20local%20variable/tailtarget.md) from talking in [Dialogue mode](../../Dialogue%20mode.md) mode and yields to the game using [waitinput](../../Global%20vars%20used/waitinput.md) as well as reset the [Text advance](../../Related%20Systems/Text%20advance.md) text skip values.
