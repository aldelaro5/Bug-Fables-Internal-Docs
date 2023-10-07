# Librarybook

Signal the game that a Lore Book has been acquired and refresh the LibraryShelf.

## Syntax

````
|librarybook|
````

## Parameters

None.

## Remarks

This command does 2 tasks:

* It increments [flagvar](../../Flags%20arrays/flagvar.md) 15 (the amount of Lore Book on the shelf).
* It calls Refresh on the first LibraryShelf that exists in the Scene.

This means that there must be at least one LibraryShelf in the Scene when this command is processed or an exception will be thrown.

This command is specifically made to handle giving a Lore Book in a place with the shelf in the same room which is normally done at the AntPalaceLibrary [maps](../../Enums%20and%20IDs/Maps.md).
