# Librarysize

An alias of [size](size.md) where the command is only processed if the pause menu is active on the "Library" window.

## Syntax

(1)

````
|librarysize,sizeuniform|
````

(2)

````
|librarysize,sizex,sizey|
````

(3)

````
|librarysize,unpause,sizex,sizey|
````

(4)

````
|librarysize,sizex,sizey,lockcontrol|
````

## Parameters

The same as [size](size.md), check its documentation to learn more.

## Remarks

Just like [size](size.md), this command affects [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by changing the horizontal size scaling used to predict the amount of space further letters will take. This is done ignoring the [size](size.md)'s locking system so even if the command isn't going to change the size once processed, it will still be taken as a width change for the auto line breaker which can cause problems.
