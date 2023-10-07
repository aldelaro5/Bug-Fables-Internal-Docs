# GetFromMap

Replace the text of this command to a dialogue line that belongs to a specific [Map](../Enums%20and%20IDs/Maps.md).

## Syntax

````
|getfrommap,map,line|
````

## Parameters

### `map`: int

The [Map](../Enums%20and%20IDs/Maps.md) id to get the dialogue line from. This must be a valid [Map](../Enums%20and%20IDs/Maps.md) id or an exception will be thrown.

### `line`: int

The line id that belongs to the specific map. This must be an int between 0 and the amount of lines the map has - 1 or an exception will be thrown.

## Remarks

The text replacement occurs with an [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the resolved line.

This command also supports the [GlobalCommand](../Related%20Systems/GlobalCommand.md) system by preprocessing the line before resuming processing at it.

Processing will resume at the same position than this command to accommodate the text replacement.
