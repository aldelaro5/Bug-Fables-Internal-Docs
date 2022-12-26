# Call

An alias of [String](String.md) where `flagstring` is a [Dialogue line id](../Dialogue%20line%20id.md) instead.

## Syntax

(1)

````
|string,lineid|
````

(2)

````
|single,lineid,clamp,maxwidth|
````

(3)

````
|string,lineid,clamp,maxwidth,widthscaleclamp|
````

(3)

````
|string,lineid,true|
````

## Parameters

The same as [String](String.md) except `flagstring` is now `lineid`:

### `lineid`:  int

The [Dialogue line id](../Dialogue%20line%20id.md) where its text will replace this command's text. This must corresponds to a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

This is one of the command supported by [Testdiag](Testdiag.md).
