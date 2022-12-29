# Sizemulti

An alias of [size](size.md) in syntax (2).

## Syntax

````
|sizemulti,sizex,sizey|
````

## Parameters

The same as [size](size.md) in syntax (2), check its documentation to learn more.

## Remarks

Just like [size](size.md), this command affects [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by changing the horizontal size scaling used to predict the amount of space further letters will take. This is done ignoring the [size](size.md)'s locking system so even if the command isn't going to change the size once processed, it will still be taken as a width change for the auto line breaker which can cause problems.
