# Area

Replace the text of this command by the current [Areas](../../../Enums%20and%20IDs/librarystuff/Areas.md)'s name.

## Syntax

````
|area|
````

## Parameters

None.

## Remarks

Specifically, the name is pulled from `Data/DialogueX/AreaNames` where X is the current [languageid](../../languageid.md).

This command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
