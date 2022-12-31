# Halfline

An alias of [Line](Line.md) where `linespacing` is 0.5.

## Syntax

````
|halfline|
````

## Parameters

None.

## Remarks

See the [Line](Line.md) page for more details on the function of the command as this one behaves the same way where `linespacing` is set to 0.5.

There is one issue specific to this command and [Quarterline](Quarterline.md) however: In [Regular Letter Rendering](../../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) and [Dialogue mode](../../Dialogue%20mode.md), this command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system twice instead of only once. This is because being in [Dialogue mode](../../Dialogue%20mode.md) no matter the letter rendering method will accumulate it before processing the command in the [SetText Life Cycle > Char loop#Vertical bar processing](../../SetText%20Life%20Cycle.md#char-loop-vertical-bar-processing), but since this command is also an alias of [Line](Line.md), it will also accumulate it a second time if the rendering method is [Regular Letter Rendering](../../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) which makes it repeat twice in a row when backtracking to the textbox containing the command.

This commend is treated like a manual line break by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it.

This is one of the command supported by [Testdiag](Testdiag.md) if the command name part is written as `line` with case sensitivity.
