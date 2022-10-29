# Sstring

Behaves like [String](String.md), but can be processed before the command is processed by the char loop. This can be done via [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) by appending the input string with the text of a flagvar slot and removing this command text. This document will detail the [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) processing. For the char loop processing, refer to the [String](String.md) command documentation.

## Syntax

````
|string,flagstring|
````

## Parameters

### `flagstring`:  int

The flagstring slot to append at the end of the input string. This must corresponds to an integer of a valid flagstring slot or an exception will be thrown.

## Remarks

This command will only get preprocessing by [OrganiseLines](../../Notable%20Methods/OrganiseLines.md)'s ReplaceFunctions's own char loop if it is called at least once with the input string before the char loop reaches this command. This command was presumably done for a similar reason than [Anstring](Anstring.md), but for the plural mark of a word which is typically adding an s at the end of the word in English. This wouldn't work completely as there are exceptions to this grammar rule.

This command is unused under normal gameplay, but it remains functional.
