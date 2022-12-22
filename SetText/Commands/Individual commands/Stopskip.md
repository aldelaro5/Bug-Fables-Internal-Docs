# Stopskip

Forces a stop on the text skip if one was in progress in [Dialogue mode](../../Dialogue%20mode.md) mode.

## Syntax

````
|stopskip|
````

## Parameters

None

## Remarks

This command sets `skiptext` to false and `isholdingskip` to [Fadeletter](Fadeletter.md). For more information on how text skip works, see [Text advance](../../Related%20Systems/Text%20advance.md).

This command does nothing in non [Dialogue mode](../../Dialogue%20mode.md) mode.
