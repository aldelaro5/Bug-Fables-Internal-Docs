# Halfline

Go to a new line below that is half the standard height and continue rendering at the start of the new line

## Syntax

````
|halfline|
````

## Parameters

None

## Remarks

This command is an alias to [Line](Line.md) sent like this:

````
|line,0.5|
````

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

This commend is treated like a manual line break by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it.

This is one of the command supported by [Testdiag](Testdiag.md) if the command name part ends with `line` with case sensitivity.
