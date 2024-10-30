# Map SetText configuration
Maps have a lot of control over how the [SetText](../SetText/SetText.md) is operated. 

Here are the configurations fields used to configure SetText:

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|useglobalcommand|bool|From prefab|Tells if the global commands system is enabled for this map. Since this system isn't complete in its implementation, this should normally not be true. See the section below for details|false|
|tattleid|int|From prefab|The SetText [dialogue line id](../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) that will be used when pressing the Help input on this map away from an NPC|0|
|englishbreakfix|bool|From prefab|If true, it changes OrganizeLines when instance.`languageid` is `English` for all SetText calls made in dialogue mode such that the fixed logic applies to fix the [whole word width line skip](../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-words-width-after-the-first-line). This is an optional fix the map needs to opt in because it can have unintended side effects as the entire English script was written with the expectation that this issue was always present|false|

## `dialogues`
The map determines the `dialogues` data to use. The value of `dialogues` is set to the result of loading a TextAsset. The location of this TextAsset is at `Data/DialoguesX/Maps/Y` where `X` is the [languageid](../SetText/languageid.md) and `Y` is the string representation of a [Maps](../Enums%20and%20IDs/Maps.md) value. By default, it's the one being initialised, but it can be changed by [readdatafromothermap](Miscellaneous%20features.md#readdatafromothermap).

There is an exception to this: if `mapid` is `TestRoom`, the data is always loaded from `Data/TestRoom`.

However, it should be noted that `dialogues` will remain null if the TextAsset couldn't be loaded. This shouldn't happen because SetText assumes there will be a value assigned to map.`dialogues`.

It should be noted that SetText has a way to read dialogue lines from other maps. It's done via the [Mapline](../SetText/Individual%20commands/GetFromMap.md) command.

## `tattleid`
All maps have what is called a "tattle" SetText lines. It is specified in the form of a regular [dialogue line id](../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) in the `tattleid` configuration field (this means it can either be a `commondialogue` line or a regular `dialogues` line).

This line is special because when pressing the HELP input on map, SetText will be called with this line. It's effectively a way to have a SetText call happen through a button press.

It is interesting to mention that due to the powerful nature of SetText, there are alternative usage other than providing help text for this feature. For example, it is possible to run conditional scripts once the HELP button is pressed such as toggling a switch which is done in the game at some point.

## `englishbreakfix`
[OrganizeLines](../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) has a major issue known as the [whole word width line skip](../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-words-width-after-the-first-line) which has a simple fix, but that fix has a bunch of bad implications over the text that was written with the assumption that this fix wouldn't exist. In other words, a lot of the `dialogues` SetText lines were initially written with the expectation that this issue always happen.

Due to this, it's not possible to apply the fix everywhere without risking to break some SetText lines. However, Maps can decide to opt-in to this fix if it won't break anything. This is what the `englishbreakfix` configuration field allows: by setting it to true, all lines processed in OrganizeLines will have the skip for this issue.

It should be noted that this feature is effectively UNUSED because the only map that has this field set to true is `SnakemouthEmpty` which isn't accessible normally and is effectively an internal test map.

## Global commands system
SetText has an unused feature called [GlobalCommand](../SetText/Related%20Systems/GlobalCommand.md). It's an incomplete attempt at addressing a flaw of SetText where dialogues lines have to be specified at the same place than their commands which can complicate localisations.

The feature is only enabled if `useglobalcommand` is set to true. When it is, Start will assign a value to `commandlines`. The value is the result of loading the TextAsset located at `Data/Commands/X` where `X` is the string representation of a [Maps](../Enums%20and%20IDs/Maps.md) value. By default, it's the one being initialised, but it can be changed by [readdatafromothermap](Miscellaneous%20features.md#readdatafromothermap).

For how `commandlines` work, check the [GlobalCommand](../SetText/Related%20Systems/GlobalCommand.md) documentation to learn more.
