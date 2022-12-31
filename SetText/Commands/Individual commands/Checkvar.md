# Checkvar

Compare a [flagvar](../../../Flags%20arrays/flagvar.md) against a value or another [flagvar](../../../Flags%20arrays/flagvar.md) for a condition and redirect if this condition ends up being true or do nothing if it ends up being false.

## Syntax

(1)

````
|checkvar,flagvar,value,lineidtrue|
````

(2)

````
|checkvar,atleast,flagvar,varvalue,lineidtrue|
````

(3)

````
|checkvar,moreand,flagvar,varvalue,lineidtrue|
````

## Parameters

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot to check. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `atleast` | `moreand`

This value determines which syntax to use. Any other value will be interpreted as `flagvar` and the command will operate in syntax (1).

* no value: redirect only if `flagvar` is equal to `value`
* `atleast`: redirect only if `flagvar` \< `varvalue`
* `moreand`: redirect only if `flagvar` >= `varvalue`
* 

### `value`: int

The value to compare the [flagvar](../../../Flags%20arrays/flagvar.md) against. This must be a valid integer or an exception will be thrown.

### `varvalue`: int | `var`int

The value to compare the [flagvar](../../../Flags%20arrays/flagvar.md) against or the second [flagvar](../../../Flags%20arrays/flagvar.md) slot to compare the first one against. 

The value form must be a valid integer or an exception will be thrown. 

The [flagvar](../../../Flags%20arrays/flagvar.md) slot form must ONLY contain the text `var` with the [flagvar](../../../Flags%20arrays/flagvar.md) slot next or before it. Any other non numerical text will be interpreted as the value form which will throw an exception. The integer part must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `lineidtrue`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to redirect to if the overall condition gets evaluated to true. If the condition gets evaluated to false, this parameter is ignored.

## Remarks

It is not possible to use syntax (1) with a [flagvar](../../../Flags%20arrays/flagvar.md) as the second operand of the condition. To do so, you can use [Checktrue](Checktrue.md) or [Checkflag](Checkflag.md)

If the applicable condition is false, this command will do nothing and processing resumes as normal.

If the applicable condition is true however, the input string will be overwritten to an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the [dialogue line id](../Dialogue%20line%20id.md) `lineidfalse` prepended with a parameterless [Blank](Blank.md) command. This will reset the character position of the [SetText Life Cycle > Char loop](../../SetText%20Life%20Cycle.md#char-loop) to restart at the beginning of the input string which will cause processing to resume at the start of the new input string. This will also disable `skiptext` if it was enabled by the [Text advance](../../Related%20Systems/Text%20advance.md) system.
