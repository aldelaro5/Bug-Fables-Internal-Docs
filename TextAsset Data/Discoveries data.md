# Discoveries data

[Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) data are split in 2 TextAssets in the game that are loaded on boot in SetVariables: 

* `Ressources/data/DiscoveryOrder`
* `Discoveries` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `DiscoveryOrder` data

The asset contains one line per [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) which contains the ordered list of the entries shown in game where each entry is one line. Each line contains 2 fields separated by `,`:
Name | Type |  Description
------- | ------- |  -------
Id | [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id | The id this line refers to
Icon | int | Index of the sprite in `Sprites/Items/EnemyPortraits`

The id is loaded into `libraryorder[0, id]`  and the icon is loaded into `discoveryicons[id]` where `id` is the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id.

The ordering is managed by PauseMenu and the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md).

## `Discoveries` language specific data

The asset contains one line per [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) whose id corresponds to the line index. Each line contains 2 fields separated by `@`:
Loaded index | Name | Type |  Description
----- | ------- | ------- |  -------
0 | Name | [SetText](../SetText/SetText.md) string | The name of the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md)
1 | Description | [SetText](../SetText/SetText.md) string | The paginated description of the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md)

The data will be loaded into `librarydata[0, id, x]` where `id` is the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id and `x` is the loaded index.

The name of the entry is handled by the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md), but the description has special handling in the pause menu. The string can contain `{` to delimit pages. The pause menu provides UI to only call [SetText](../SetText/SetText.md) with the current page's text and a way to browse the different pages. 

It is also possible to conditionally decide to not render further pages by using `}flags}` as delimiter instead where `flags` is a [flags](../Flags%20arrays/flags.md) slot that must be true to process any further pages. If it's false, the preceding page of the corresponding delimiter will be the last processed page. This allows to render further details only after progress has been done in the game.
