# Unpausesize

An alias of [size](size.md) where the command is only processed if the pause menu and the questboardobj for the [Quest Board List Type](../../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) are inactive.

## Syntax

(1)

````
|unpausesize,sizeuniform|
````

(2)

````
|unpausesize,sizex,sizey|
````

(3)

````
|unpausesize,unpause,sizex,sizey|
````

(4)

````
|unpausesize,sizex,sizey,lockcontrol|
````

## Parameters

The same as [size](size.md), check its documentation to learn more.

## Remarks

Just like [size](size.md), this command affects [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by changing the horizontal size scaling used to predict the amount of space further letters will take. This is done ignoring the [size](size.md)'s locking system so even if the command isn't going to change the size once processed, it will still be taken as a width change for the auto line breaker which can cause problems.
