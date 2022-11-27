# Checkflag

A variation of [Checktrue](Checktrue.md) where the only difference is the flags condition are inverted (there are no changes to flagvar checks). See the remarks section of this page for the exact logic.

## Syntax

(1)

````
|Checkflag,flags,lineidfalse|
````

(2)

````
|Checkflag,var,flagvar,value,lineidfalse|
````

## Parameters

The same as [Checktrue](Checktrue.md).

## Remarks

In its simplest form, this command will evaluate the following:

* If syntax (1) and `flags` is one flags slot, check if that slot is false
* If syntax (1) and `flags` is an array of integer, for each int:
  * if it is above 0, check if the corresponding flags slot is false
  * If it is 0 or below, take the absolute value and check if the corresponding flags slot is true
* If Syntax (2), check that the flagvar slot `flagvar` is NOT equal to `value`

If all the applicable conditions are true, this command will do nothing and processing resumes as normal.

If any of the applicable conditions is false however, the input string will be overwritten to an [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) version of the dialogue line at id `lineidfalse`. This will reset the character position of the [life cycle > Char loop](../../life%20cycle.md#char-loop) to restart at the beginning of the input string which will cause processing to resume at the start of the new input string. This will also disable `skiptext` if it was enabled by the [Text advance](../../Related%20Systems/Text%20advance.md) system.
