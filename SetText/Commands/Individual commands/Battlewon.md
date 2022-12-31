# Battlewon

Do nothing if the last battle or card battle was won, but redirect to a different dialogue line if it was lost or fled from.

## Syntax

(1)

````
|battlewon,lineidlost|
````

(2) (May throw an exception, not recommended, see remarks)

````
|checktrue,flags,lineidfalse|
````

(3) (Will throw an exception, see remarks)

````
|battlewon,var,flagvar,value,lineidfalse|
````

## Parameters

For syntax (2) and (3), the parameters are the same then [Checkflag](Checkflag.md) and [Checktrue](Checktrue.md).

### `lineidlost`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to redirect to if the last battle or card battle was lost or fled from. In such cases, this must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown. Otherwise, this parameter is ignored.

## Remarks

While syntax (2) and (3) are technically possible, `flags` and `var` will get interpreted as `lineidlost`. This means (3) will always throw an exception while (2) will throw if it happens to not correspond to a valid [Dialogue line id](../Dialogue%20line%20id.md).

If the last battle or card battle was won, this command will do nothing and processing resumes as normal.

If the last battle was lost or fled from however, the input string will be overwritten to an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue line at id `lineidlost`. This will reset the character position of the [SetText Life Cycle > Char loop](../../SetText%20Life%20Cycle.md#char-loop) to restart at the beginning of the input string which will cause processing to resume at the start of the new input string. This will also disable `skiptext` if it was enabled by the [Text advance](../../Related%20Systems/Text%20advance.md) system.
