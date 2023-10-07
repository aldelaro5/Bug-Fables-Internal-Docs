# Dialogue line ID

[SetText](../SetText.md) commands often uses the same format for specifying a dialogue line id from the current map or from CommonDialogue which is an int value.

## Current map line id

Any value >= 0 is considered a current map line id. This means that it will check the current map, resolve its enum value name and pick the file of the same name inside the `maps` folder of the current language's dialogue folder. The value directly corresponds to a line index starting from 0. If no such line exists, an exception will be thrown.

## CommonDialogue line id

To refer to a CommonDialogue line id instead of a current map line id, the value must be specified in the form `-id - 1`. Any value below 0 will be interpreted as a CommonDialogue line id with the id being the absolute version of the value - 1. For example, -1 refers to CommonDialogue line 0. The file is always the one named `CommonDialogue` located at the root of the current language's dialogue folder. If no such line exists, an exception will be thrown.

## What about MenuText?

This format does not cover MenuText. To use menu text, it must be done by code, using a [menu](Individual%20commands/Menu.md) or [prompt](Individual%20commands/Prompt.md) command which supports specifying one.
