# Sstring

Alias to [String](String.md) if processed by SetText directly, but it can also be processed before on [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by replacing the text of this command to a [flagstring](../../../Flags%20arrays/flagstring.md). For the char loop processing, refer to the [String](String.md) command documentation.

## Syntax

````
|string,flagstring|
````

## Parameters

### `flagstring`:  int

The [flagstring](../../../Flags%20arrays/flagstring.md) slot to append at the end of the input string. This must corresponds to an integer of a valid [flagstring](../../../Flags%20arrays/flagstring.md) slot or an exception will be thrown.

## Remarks

This command can get processed by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s ReplaceFunctions's own char loop if it is called at least once with the input string before the char loop reaches this command.

This command is unused under normal gameplay, but it remains functional.
