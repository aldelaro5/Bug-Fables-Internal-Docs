# Noskip

Forces a stop on the text skip if one was in progress and also prevents any text skip to occur from this point until it is allowed again in [Dialogue mode](../../Dialogue%20mode.md) mode.

## Syntax

````
|noskip|
````

## Parameters

None

## Remarks

This commands does the same actions then [Stopskip](Stopskip.md) with the addition that it sets `Noskip` to false which prevents further text skip. It also sets `isholdingskip` to [Fadeletter](Fadeletter.md). For more information, see [Text advance](../../Related%20Systems/Text%20advance.md).

This command does nothing in non [Dialogue mode](../../Dialogue%20mode.md) mode.
