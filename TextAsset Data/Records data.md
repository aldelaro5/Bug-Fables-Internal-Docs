# Records data

[Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) data are split in 2 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/SynopsisOrder`
* `Synopsis` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `SynopsisOrder` data

The asset contains one line per [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) which contains the ordered list of the entries shown in game where each entry is one line. Each line contains 2 fields separated by `,`:
Name | Type |  Description
------- | ------- |  -------
Id | [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) id | The id this line refers to
Icon | int | Index of the sprite in `Sprites/Items/EnemyPortraits`

The id is loaded into `libraryorder[3, id]`  and the icon is loaded into `achiveicons[id]` where `id` is the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id.

The ordering is managed by PauseMenu and the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md).

## `Synopsis` data

The asset contains one line per [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) whose id corresponds to the line index. Each line contains 2 fields separated by `@`:
Loaded index | Name | Type |  Description
----- | ------- | ------- |  -------
0 | Name | [SetText](../SetText/SetText.md) string | The name of the [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md)
1 | Description | [SetText](../SetText/SetText.md) string | The description of the [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md)

The data will be loaded into `librarydata[2, id, x]` where `id` is the [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) id and `x` is the loaded index.

The name of the entry is handled by the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md) while the description is only rendered underneath the icon in the PauseMenu.
