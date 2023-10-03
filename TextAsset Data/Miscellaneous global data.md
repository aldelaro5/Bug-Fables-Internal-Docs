# Miscellaneous global data

This page contains documentations about some minor [languageid](../SetText/languageid.md) agnostic data the game uses.

## Language prompt help lines

`LanguageHelp` is a TextAsset in `Ressources/data` loaded on boot by LoadEssentials into the `languagehelp` array. Each lines contains the prompt text to render during the [languageid](../SetText/languageid.md) selection whose line corresponds to the selected [languageid](../SetText/languageid.md). This is meant to be refreshed every time the selection changes which is handled by the [Languages list Type](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md).

## LetterPrompts

There are several TextAssets in `Ressources/data` whose name starts with `LetterPrompt` followed by the corresponding letter prompt id. Each of them corresponds to the text shown in a [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md) command depending on the prompt presented. The data gets loaded in the course of the command on `CreateLetterPrompt`, but also on boot by `FontSet`. This is because due to the way TextMesh data gets cached on Unity, the game will reserve the first letter slot permanently on boot by doing a [SetText](../SetText/SetText.md) call in non [Dialogue mode](../SetText/Dialogue%20mode.md) where the text is every letter prompt data in a row, one for each [fonttype](../SetText/fonttype.md) all in [Single Letter Rendering](../SetText/Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md). This very large string forces Unity to put the text meshes data into its cache which improves performance.

For more information on each letter prompt id, consult the details of the [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md) commands. The files are named like so:

* `LetterPrompt0`
* `LetterPrompt1`
* `LetterPrompt2`
* `LetterPrompt3`
* `LetterPrompt4`

## Leaves destination positions during battle transition

`LeafPos` is a TextAsset in `Ressources/data` loaded on boot by LoadEssentials into the `leafpos` array that tells where the leaves in the standard battle transitions will end up on the screen once they are done moving. While their starting position and final rotation are random, their destination position aren't and they are determined by this data. It is possible to add or remove leaves by changing the amount of lines as each leaf corresponds to one line.

## Unused

There are some unused TextAsset in the game located in `Ressources/data` that are never loaded by the game, but the data is still present:

* `boardgame/1data`: This is presumably a remnant of the unused [PartyGame](../SetText/Commands/Individual%20commands/PartyGame.md) which had a SetText commands and other assets left. It is unknown how the layout of the data was layed out, but it seems this represented the data of a board whose id was 1.
* `commands/SnakemouthEmpty`: This is a remnant of the unused [GlobalCommand](../SetText/Related%20Systems/GlobalCommand.md) feature.
* `BoardData`: Seems to be similar to `boardgame/1data`, but the placement of the asset suggests it was intended to refer to every board. The layout is unknown.
* `CookIcons`: It seems to be an earlier version of [Recipes data](Recipes%20data.md) as the name and the contents suggests it was intended to contain sprite index information as well as a custom description. The final version of the game ended up determining both of them dynamically from the [Item](../Enums%20and%20IDs/Items.md) associated with the recipes.
* `MapOrder`: It seems to be an early version of an ordering system similar to the other library pages, but for the areas. It only contains the numbers from 0 to 10 in order which would indicate the file became redundant. It ended up being unneeded anyway because the game never needs to present an [ItemList](../ItemList/ItemList.md) of [Areas](../Enums%20and%20IDs/librarystuff/Areas.md).
* `Letters`: It seems to be an earlier version of the optimization mentioned above about LetterPrompt data where the text was directly specified in its own asset.
* `FortuneData`: It seems to be an earlier version of [Fortune Teller data](Fortune%20Teller%20data.md) before this information ended up being hardcoded.
