# Miscellaneous language specific data

This page contains documentations about some minor [languageid](../SetText/languageid.md) specific data the game uses. Each TextAsset mentioned is present in the corresponding language's dialogue directory.

# Action commands instructions

`ActionCommands` is a TextAsset loaded on boot by SetVariables into the `commandhelptext` array that contains instructions to perform an action command in battle. Each line corresponds to one action command refered by its id. TODO: document what each id means exactly.

# Areas

There are 2 TextAsset that describes each [Areas](../Enums%20and%20IDs/librarystuff/Areas.md): `AreaNames` and `AreaDesc`. `AreaNames` is loaded on boot by SetVariables into the `areanames` array and it contains an area name whose line index corresponds to an [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) id. `AreaDesc` is loaded by PauseMenu's MapSetup into the `enemydata` array and it contains an area description whose line index corresponds to an [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) id.

# Credits

`Credits` contains the text rendered in the credits which has its own rendering scheme. It is loaded on [Event](../SetText/Commands/Individual%20commands/Event.md) 204 (credits) and it gets rendered using lists of TextMesh in rich text mode. It is the only notable textual information not rendered using [SetText](../SetText/SetText.md).

# Unused

There is one unused TextAsset called `BoardGame` which presumably was a remnant of the unused [PartyGame](../SetText/Commands/Individual%20commands/PartyGame.md) which had its own command. It presumably acted similarly to `CardGame` where it would contain [SetText](../SetText/SetText.md) lines used for the game only.

TODO: there's a LibraryMap.asset ???
