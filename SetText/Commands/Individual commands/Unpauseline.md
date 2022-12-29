# Unpauseline

An alias of [Line](Line.md) that is only processed by SetText when the pause menu is not active.

## Syntax

(1)

````
|unpauseline|
````

(2)

````
|unpauseline,linespacing|
````

## Parameters

The same as [Line](Line.md). Check its documentation to learn more.

## Remarks

Just like [Line](Line.md), this commend is treated like a manual line break by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it, but it only does so if the pause menu is inactive. If it is active, this will not be treated like a logical line break.

This command was presumably done as a workaround to the [OrganiseLines Known Issues > Not counting a whole word's width after the first line](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-word-s-width-after-the-first-line) problem.
