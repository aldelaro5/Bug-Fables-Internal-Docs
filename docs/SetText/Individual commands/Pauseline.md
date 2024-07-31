# Pauseline

An alias of [line](Line.md) that is only processed by SetText when the pause menu is active and either in no window or in the following windows: top level window, "Inventory" and "Medals and Stats".

## Syntax

(1)

````
|pauseline|
````

(2)

````
|pauseline,linespacing|
````

## Parameters

The same as [line](Line.md). Check its documentation to learn more.

## Remarks

Just like [line](Line.md), this commend is treated like a manual line break by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it.

This command was presumably done as a workaround to the [OrganiseLines Known Issues > Not counting a whole word's width after the first line](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-words-width-after-the-first-line) problem.
