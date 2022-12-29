# Battlesize

An alias of [size](size.md) or [Sizemulti](Sizemulti.md) that is only processed if a battle is in progress.

## Syntax

(1)

````
|battlesize,sizeuniform|
````

(2)

````
|battlesize,sizex,sizey|
````

(3)

````
|battlesize,unpause,sizex,sizey|
````

(4)

````
|battlesize,sizex,sizey,lockcontrol|
````

(5) (Alias of [Sizemulti](Sizemulti.md))

````
|battlesize,multi,sizex,sizey|
````

## Parameters

The same as [size](size.md) for syntax (1) to (4), check its documentation to learn more. For syntax (5), the same as [Sizemulti](Sizemulti.md), check its documentation to learn more and the `multi` parameter must be sent to indicate using this syntax.

## Remarks

This command does nothing if a battle isn't in progress.

Unlike [size](size.md), this command does nothing in [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) because it is explicitly excluded.
