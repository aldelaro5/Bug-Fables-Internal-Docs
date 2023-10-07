# Call

An alias of [string](String.md) where `flagstring` is a [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) instead.

## Syntax

(1)

````
|call,lineid|
````

(2)

````
|call,lineid,clamp,maxwidth|
````

(3)

````
|call,lineid,clamp,maxwidth,widthscaleclamp|
````

(3)

````
|call,lineid,true|
````

## Parameters

The same as [string](String.md) except `flagstring` is now `lineid`:

### `lineid`:  int

The [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) where its text will replace this command's text. This must corresponds to a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

This is one of the command supported by [testdiag](Testdiag.md).
