# Library list type

Display a list of the Library content with the page being [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), [Bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md), [Recipes entry](../../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) or [Records entry](../../Enums%20and%20IDs/librarystuff/Records%20entry.md).

## Options generation

`listvar` is all the ids matching the page of the list ordered by their respective order from the game's data.

`listredirect` is overridden to null.

## Option's SetText input string

First, the main text is set to the string representation of the option + 1 padded by 3 `0` followed by `-`.

Then, the corresponding name of the library entry is appended to the string.

If the language is set to `German` and the list is the [Discoveries entry](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), |`sizemulti`,0.7,1| is prepended to the string.

If the language is set to `Japanese` and we are paused, |`size`,0.75,y,[lock](../../SetText/Commands/Individual%20commands/Lock.md)\| is prepended to the string.

Finally, |[single](../../SetText/Commands/Individual%20commands/Single.md)\| is prepended to the string which will serve as the final string.

The x position of the text is overridden to -0.5 and the size to Vector2.one.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. This list type does not have any behavior defined for this phase.
