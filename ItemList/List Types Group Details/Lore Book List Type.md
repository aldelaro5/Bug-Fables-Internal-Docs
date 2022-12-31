# Lore book list type

Display the list of the unlocked lore books.

## Options generation

`listvar` is all the integers from 0 to the [flagvar](../../Flags%20arrays/flagvar.md) 15 subtracted by 1. In practice, this represents the numbers from 0 to the amount of lore book unlocked - 1.

`listredirect` is overridden to -66 which, corresponds to the CommonDialogue line 65 and has the following: |[tail](../../SetText/Commands/Individual%20commands/Tail.md),null||[boxstyle](../../SetText/Commands/Individual%20commands/Boxstyle.md),3||[Bleep](../../SetText/Commands/Individual%20commands/Bleep.md),2,1,1||[Lore](../../SetText/Commands/Individual%20commands/Lore.md)\|.

## Option's [SetText](../../SetText/SetText.md) input string

The text is the name of the lore book obtained from LoreText. If the option is `option`, [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the text of the lore book from LoreText.

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the selected option (bug?). Then, the list gets destroyed which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).
