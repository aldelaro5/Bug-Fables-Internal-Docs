# Checktrue

Check a [flagvar](../../../Flags%20arrays/flagvar.md) for NOT being a specific integer, a single [flags](../../../Flags%20arrays/flags.md) for being true or a set of [flags](../../../Flags%20arrays/flags.md) for a specific state and either do nothing if all conditions are satisfied or redirect to a different line if they aren't.

## Syntax

(1)

````
|checktrue,flags,lineidfalse|
````

(2)

````
|checktrue,var,flagvar,value,lineidfalse|
````

## Parameters

### `flags`: int | array split by `@`: int

The [flags](../../../Flags%20arrays/flags.md) slot or array of [flags](../../../Flags%20arrays/flags.md) slot to check for being true. It is possible to only specify one slot by omitting any `@`. 

In the singular form, the value must be a value flags slot or an exception will be thrown. In this form, only checking for a single flag being true is supported.

In the array form, each element of the array must be have its absolute value be a valid flags slot or an exception will be thrown. Any negative version of a slot or `0` will instruct to check for the flag being false instead of true.

### `var`

This value determines whether to operate in syntax (2) or not. Any other value will be interpreted as `flags`.

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot to check. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `value`: int

The value to compare the [flagvar](../../../Flags%20arrays/flagvar.md) against for inequality. This must be a valid integer or an exception will be thrown.

### `lineidfalse`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to redirect to if the overall condition gets evaluated to false. If the condition gets evaluated to true, this parameter is ignored.

## Remarks

In its simplest form, this command will evaluate the following:

* If syntax (1) and `flags` is one [flags](../../../Flags%20arrays/flags.md) slot, check if that slot is true
* If syntax (1) and `flags` is an array of integer, for each int:
  * if it is above 0, check if the corresponding [flags](../../../Flags%20arrays/flags.md) slot is true
  * If it is 0 or below, take the absolute value and check if the corresponding [flags](../../../Flags%20arrays/flags.md) slot is false
* If Syntax (2), check that the [flagvar](../../../Flags%20arrays/flagvar.md) slot `flagvar` is NOT equal to `value`

If any the applicable conditions are false, this command will do nothing and processing resumes as normal.

If all of the applicable conditions are true however, the input string will be overwritten to an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue line at id `lineidfalse`. This will reset the character position of the [SetText Life Cycle > Char loop](../../SetText%20Life%20Cycle.md#char-loop) to restart at the beginning of the input string which will cause processing to resume at the start of the new input string. This will also disable `skiptext` if it was enabled by the [Text advance](../../Related%20Systems/Text%20advance.md) system.
