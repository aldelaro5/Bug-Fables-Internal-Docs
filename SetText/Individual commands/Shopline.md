# Shopline

An alias of [line](Line.md) that is only processed by SetText when unpaused and the player has control or an [ItemList](../../ItemList/ItemList.md) is getting processed.

## Syntax

(1)

````
|shopline|
````

(2)

````
|shopline,linespacing|
````

## Parameters

The same as [line](Line.md). Check its documentation to learn more.

## Remarks

Just like [line](Line.md), this commend is treated like a manual line break by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) since it logically resets the line to the first one and goes at the start of it. However, this is only applicable if [ItemList State Machine](../../ItemList/ItemList%20State%20Machine.md)'s `inlist` is true (see [inlist issue](../../ItemList/inlist%20issue.md)) or if the player is interacting with an entity with an interaction of type Shop. If neither of these are true, this won't be treated as a line break.

This command was presumably done as a workaround to the [OrganiseLines Known Issues > Not counting a whole word's width after the first line](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-word-s-width-after-the-first-line) problem.
