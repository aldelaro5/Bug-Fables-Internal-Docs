# Stat Bonuses List Type

Display the list of all the stat bonuses applied.

## Options generation

`listvar` has one element being -1 if there are no statbonus in the array.

Otherwise, `listvar` is all the bonus types of the statbonus in order. The amount and target are stored in a separate local 2 dimensional array where each element contains the amount and type respectively.

## Option's [SetText](../../SetText/SetText.md) input string

If the option is -1, the text is |[color](../../SetText/Individual%20commands/Color.md),1| followed by MenuText line 20 which is "No Items" in English and [flagvar](../../Flags%20arrays/flagvar.md) 0 is set to -1.

Otherwise, the text is set to the text representation of the StatBonus's name followed by ` x` followed by the amount stored locally followed by `to` followed by the target's name stored locally. The target name is determined like the following: if it's -1, it's `All` and if it isn't, it's MenuText line 46 + the target which are Team Snakemouth's names in order. [flagvar](../../Flags%20arrays/flagvar.md) 1 is also set to `option`.

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty.

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the selected option (bug?). Then, the list gets destroyed which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).

## Remarks

This list is not available under normal gameplay because it is only used in the TestRoom.
