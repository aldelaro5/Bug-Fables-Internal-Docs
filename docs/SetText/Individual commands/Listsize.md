# Listsize

An alias of [size](size.md) or [sizemulti](Sizemulti.md) that is only processed if a battle is in progress, the pause menu is inactive and we are in an [ItemList](../../ItemList/ItemList.md).

## Syntax

(1)

````
|listsize,sizeuniform|
````

(2)

````
|listsize,sizex,sizey|
````

(3)

````
|listsize,unpause,sizex,sizey|
````

(4)

````
|listsize,sizex,sizey,lockcontrol|
````

(5) (Alias of [sizemulti](Sizemulti.md))

````
|listsize,multi,sizex,sizey|
````

## Parameters

The same as [size](size.md) for syntax (1) to (4), check its documentation to learn more. For syntax (5), the same as [sizemulti](Sizemulti.md), check its documentation to learn more and the `multi` parameter must be sent to indicate using this syntax.

## Remarks

This command does nothing if a battle isn't in progress, the pause menu is active or we aren't in an [ItemList](../../ItemList/ItemList.md).

Unless syntax (5) is used, just like [size](size.md), this command affects [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by changing the horizontal size scaling used to predict the amount of space further letters will take. This is done ignoring the [size](size.md)'s locking system so even if the command isn't going to change the size once processed, it will still be taken as a width change for the auto line breaker which can cause problems. 

If syntax (5) is used however, this command does nothing in [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md).
