# Common

Replace the input string to a parameterless [blank](Blank.md) command followed by a line in commondialogue and continue processing at the start of it.

## Syntax

````
|common,commonlineindex|
````

## Parameters

### `commonlineindex`: int

The index of the commondialogue line to redirect to. Unlike a regular [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md), this must be a valid index of commondialogue or an exception will be thrown.

## Remarks

The actual line goes through [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) before the [blank](Blank.md) command is prepanded and that string ends up being the new input string.

This is essentially a stripped down version of [goto](Goto.md) which can do the same thing, but with [Backtracking](../Related%20Systems/Backtracking.md), [testdiag](Testdiag.md) and more. This command was presumably only used for debugging purposes.

This command resumes processing at the start of the string to accommodate the new input string.
