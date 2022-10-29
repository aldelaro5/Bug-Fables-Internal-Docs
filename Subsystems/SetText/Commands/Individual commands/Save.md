# Save

Attempts to save the game and continue processing with an output message.

## Syntax

````
|save|
````

## Parameters

None.

## Remarks

This command does the following:

* Disables the ability for the GC and unused resources to be cleared.
* Yields a frame.
* Attempts to save using the `caller`'s first vectordata as the starting position saved on the file.
* On Success:
  * Play the AtkSuccess sound effect at 60% volume.
  * Replaces the input string with an [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) version of  the save success message from MenuText with a `|blank|` prepended to it and a `|stopskip|` appended to it.
* On failure:
  * Replaces the input string with an [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) version of  the save failed message from MenuText with a `|blank|` prepended to it and a `|stopskip|` appended to it.
* Yields for a frame.
* Restores the ability for the GC and unused resources to be cleared.

This command will cause SetText to resume processing at the start position to accommodate the text replacement of the input string.
