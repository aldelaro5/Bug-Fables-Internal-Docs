# Goto

Replace the input string by a particular dialogue line or a randomly selected one from a list and continue processing at the start of the new string. This has support for appending commands and to not blank the textbox before the replacement.

## Syntax

(1)

````
|goto,line|
````

(2)

````
|goto,line,keep|
````

(3)

````
|goto,line,commands...|
````

(4)

````
|goto,random,lines...|
````

## Parameters

### `line`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to replace the input string with. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

### `commands`: `keep` | string...

When the value is `keep`, this instruct the command to not prepend a |[blank](Blank.md)\| to the new input string. Any other value will be interpreted as a variable list of [Commands](../Commands.md) syntax string without the delimiting `|`s. The `|`s must be omitted in the values as this command will add them during parameter parsing and failure to omit them in the values will break the command parser which will cause undefined behaviors. Refer to each command's syntax for how to properly form a valid command string for each values.

### `random`

Specifies to randomly select the line from the `lines` list with a uniform probability distribution. Any other value will be interpreted as the `line` parameter which will cause an exception to be thrown.

### `lines`: int...

The list of dialogue lines to randomly choose from for (4). Each value must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

After the new input string is obtained, it will append in order the commands defined in `commands` if it's not `keep`. It will then call [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) on the result and prepend a |[blank](Blank.md)\| on the organised string if `commands` was not `keep`. The result will become the new input string.

After this command is done, processing will resume at the start of the new input string.

This is one of the 2 commands alongside [Next](Next.md) that allows to delimit the textbox's text in the [Backtracking](../../Related%20Systems/Backtracking.md) system no matter the syntax form.

This is one of the command supported by [Testdiag](Testdiag.md).
