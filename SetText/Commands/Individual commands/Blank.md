# Blank

Clears the current textbox by freeing all letter slots and go at the start of the first line.

## Syntax

````
|blank|
````

## Parameters

The parameters corresponds to the ones sent to a [Tail](Tail.md) command at the end of this command's processing. If no parameters are sent, the [Tail](Tail.md) command isn't processed.

## Remarks

This also clears up the icons and sprites accumulated in the current box and resets the text color to black. It also resets the `tempdiag` accumulator from the [Backtracking](../../Related%20Systems/Backtracking.md) system to |`size`,size.x,size.y| using the `Size` values. At the end of this command, if there were parameters sent, a [Tail](Tail.md) command will be processed with them at the end of this command's processing.

This commend is treated like a manual line break by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it.
