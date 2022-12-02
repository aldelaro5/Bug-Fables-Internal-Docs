# [Medal](../../Enums%20and%20IDs/Medal.md)s list type

Display the list of [medal](../../Enums%20and%20IDs/Medal.md)s currently in possession, not to be confused with [Equipped Medals List Type](Equipped%20Medals%20List%20Type.md).

## Options generation

`listvar` will be all the numbers from 0 to the amount of [medal](../../Enums%20and%20IDs/Medal.md)s in possession - 1.

If paused, `listredirect` is overridden to null.

## Option's SetText input string

If there is no options or the option is -1, the string will be |[color](../../SetText/Commands/Individual%20commands/Color.md),1| followed by the no medals text from MenuText line id 23. Nothing else is done once the string is assigned in this case. It should be noted that if there is indeed no options, this code will never be run because `maxoptions` will also report not having anything so this phase will not have any iterations.

Otherwise, the input string will be |[single](../../SetText/Commands/Individual%20commands/Single.md)\| + `  ` followed by the medal's name obtained from badgedata whose medal id is obtained from looking at the medal the player has located at the index of the option. From there a sprite of the medal is rendered on the rightmost part of the bar with different properties whether we are paused or not:

* When unpaused, localPosition is (-2.5, 0.0) and localScale is (0.55, 0.6, 1.0)
* When paused, localPosition is (-0.25, 0.0) and localScale is (0.7, 0.7, 1.0)

Additionally, there are different additional behaviors depending if we are paused or not:

* When unpaused, the y position of the main text is overridden to -0.2.
* When paused, the position of the main text is overridden to (0.35, -0.3), the size to Vector2.one and the bar height used for calculating the y position of the down arrow is overridden to -0.9. There is also going to be a bar rendered at the background if the medal is equipped in any way. The color is FFC000 for Vi, 00B800 for Kabbu, 00AFE6 for Leif and FFAE00 for equipped to all. If the medal is equipped to a character, its character icon sprite will also be rendered towards the right of the bar.

Finally, if this isn't a `sell` list, render the MP icon and the MP cost towards the right of the bar. The cost part is done via [SetText](../../SetText/SetText.md) in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is |`font`,0| followed by the cost obtained from badgedata clamped from 0 to 999 or from 0 to 1 if the RUIGEE [flags](../../Flags%20arrays/flags.md) is active.

## Description box rendering

Nothing is rendered if the option is -1.

Otherwise, It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the medal's description obtained from badgedata using the id of the [medal](../../Enums%20and%20IDs/Medal.md) the player has located at the index of the option. If this is a `sell` list, the secondary string is the medal's name obtained in a similar manner.
