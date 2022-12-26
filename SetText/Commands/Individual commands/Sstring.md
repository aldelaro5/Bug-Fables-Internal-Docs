# Sstring

Alias to [String](String.md) if processed by SetText directly, but it can also be processed before on [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by replacing the text of this command to a [flagstring](../../../Flags%20arrays/flagstring.md). For the char loop processing, refer to the [String](String.md) command documentation.

## Syntax

(1)

````
|sstring,flagstring|
````

(2)

````
|sstring,flagstring,clamp,maxwidth|
````

(3)

````
|sstring,flagstring,clamp,maxwidth,widthscaleclamp|
````

(3)

````
|sstring,flagstring,true|
````

## Parameters

The same as [String](String.md).

## Remarks

This command can get processed by [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s ReplaceFunctions's own char loop if it is called at least once with the input string before the char loop reaches this command. This only supports syntax (1) however and any other syntax will be processed as if it was syntax (1). The other syntaxes are only supported if SetText processes the command before [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md).

This command is unused under normal gameplay, but it remains functional.
