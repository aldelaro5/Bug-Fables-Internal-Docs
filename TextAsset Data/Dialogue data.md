# Dialogue data

The dialogue lines are present in some TextAssets which are either global to the game or belong to a specific [Maps](../Enums%20and%20IDs/Maps.md). The lines are however language specific with the exception of the TestRoom map which is language agnostic and the data is inside the `TestRoom` asset. Each lines contains a single [SetText](../SetText/SetText.md) string and the line index is how the game can access it (they are generally hardcoded or referenced somewhere else like in asset data).

In `Ressources/data`, there are multiple directories named `dialogueX` where X is the corresponding [languageid](../SetText/languageid.md). All the data belonging specifically to that language are inside. The different files are documented in each of their respective pages, but this one will focus more on the dialogue lines aspect; the ones notably used in [Events](../Enums%20and%20IDs/Events.md), UI and others.

## Global dialogue data

There are 2 files that are notable for containing global language specific data: `CommonDialogue` and `MenuText`.

### Common dialogue lines

Some [SetText](../SetText/SetText.md) lines are shared across [Maps](../Enums%20and%20IDs/Maps.md) in the game and needs global access via the standard [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) system. These lines are located in the `CommonDialogue` TextAsset which is loaded on boot by SetVariables into `commondialogue`. It is also possible to access a line using the [Common](../SetText/Commands/Individual%20commands/Common.md) command.

### Menu dialogue lines

`MenuText` behaves similarly to `CommonDialogue`, but the lines cannot be resolved using a standard [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) as each [SetText](../SetText/SetText.md) [Commands](../SetText/Commands/Commands.md) have to specifically support them. They are mainly intended for UI rendering. The asset is loaded on boot by SetVariables into `menutext`. The main way to use a line directly via a command is by using [Menu](../SetText/Commands/Individual%20commands/Menu.md).

## [Maps](../Enums%20and%20IDs/Maps.md) specific dialogue lines

The `maps` directory in each dialogue directory contains several TextAsset, each named after a [Map](../Enums%20and%20IDs/Maps.md)'s name and it contains dialogue lines specific to that map. This means that under normal operation, [SetText](../SetText/SetText.md) commands can only access the lines via the standard [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) system if the current map matches the file the line is in. There is however a way to load a line from another map using the [GetFromMap](../SetText/Commands/Individual%20commands/GetFromMap.md) command.
