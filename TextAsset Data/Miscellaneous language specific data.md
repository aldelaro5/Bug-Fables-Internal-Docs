# Miscellaneous language specific data

This page contains documentations about some minor [languageid](../SetText/languageid.md) specific data the game uses. Each TextAsset mentioned is present in the corresponding language's dialogue directory.

# Action commands instructions

`ActionCommands` is a TextAsset loaded on boot by SetVariables into the `commandhelptext` array that contains instructions to perform an action command in battle. Each line corresponds to one action command refered by its id. TODO: document what each id means exactly.

# Areas

There are 2 TextAsset that describes each [Areas](../Enums%20and%20IDs/librarystuff/Areas.md): `AreaNames` and `AreaDesc`. `AreaNames` is loaded on boot by SetVariables into the `areanames` array and it contains an area name whose line index corresponds to an [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) id. `AreaDesc` is loaded by PauseMenu's MapSetup into the `enemydata` array and it contains an area description whose line index corresponds to an [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) id.

# Common dialogue lines

Some [SetText](../SetText/SetText.md) lines are shared across [Maps](../Enums%20and%20IDs/Maps.md) in the game and needs global access via the standard [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) system. These lines are located in the `CommonDialogue` TextAsset which is loaded on boot by SetVariables into `commondialogue`.

# Credits

`Credits` contains the text rendered in the credits which has its own rendering scheme. It is loaded on [Event](../SetText/Commands/Individual%20commands/Event.md) 204 (credits) and it gets rendered using lists of TextMesh in rich text mode. It is the only notable textual information not rendered using [SetText](../SetText/SetText.md).

# Menu dialogue lines

`MenuText` behaves similarly to `CommonDialogue`, but the lines cannot be resolved using a standard [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) as each [SetText](../SetText/SetText.md) [Commands](../SetText/Commands/Commands.md) have to specifically support them. They are mainly intended for UI rendering. The asset is loaded on boot by SetVariables into `menutext`.

# Unused

There is one unused TextAsset called `BoardGame` which presumably was a remnant of the unused [PartyGame](../SetText/Commands/Individual%20commands/PartyGame.md) which had its own command. It presumably acted similarly to `CardGame` where it would contain [SetText](../SetText/SetText.md) lines used for the game only.

TODO: there's a LibraryMap.asset ???
